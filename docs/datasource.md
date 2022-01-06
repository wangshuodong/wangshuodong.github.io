> 抽象类获取连接

```java
@Data
public abstract class MyAbstractDataSource implements DataSource {

    private String url;
    private String driver;
    private String username;
    private String password;
    //最大的正在使用的连接数
    private int maxActiveConnections = 10;
    //最大空闲连接数
    private int maxIdleConnections = 10;
    //从连接池中获取一个连接最大等待时间
    private int poolTimeWait = 30000;

    @Override
    public Connection getConnection() throws SQLException {
        return getConnection(username, password);
    }

    @Override
    public Connection getConnection(String username, String password) throws SQLException {
        Connection connection = DriverManager.getConnection(url, username, password);
        return connection;
    }
}
```
> 代理类，关闭的时候把连接放入连接池
```java
@Data
public class ConnectionProxy implements InvocationHandler {
    //真实连接
    private Connection realConnection;
    //代理连接
    private Connection proxyConnection;
    //数据源对象
    private MyDataSource myDataSource;

    public ConnectionProxy(Connection realConnection, MyDataSource myDataSource) {
        this.realConnection = realConnection;
        this.myDataSource = myDataSource;
        //初始化代理连接
        this.proxyConnection = (Connection) Proxy.newProxyInstance(
                realConnection.getClass().getClassLoader(),
                realConnection.getClass().getInterfaces(),
                this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        if ("close".equalsIgnoreCase(methodName)) {
            //把连接放入连接池
            myDataSource.colseConnection(this);
        } else {
            method.invoke(realConnection, args);
        }
        return null;
    }
}
```
> 自己模拟的数据库连接池类
```java
public class MyDataSource extends MyAbstractDataSource {
    //空闲连接池
    private final List<ConnectionProxy> idleConnections = new ArrayList<>();
    //空闲连接池
    private final List<ConnectionProxy> activeConnections = new ArrayList<>();
    //监视器对象
    private final Object monitor = new Object();

    @Override
    public Connection getConnection() throws SQLException {
        ConnectionProxy connectionProxy = getConnectionProxy(super.getUsername(), super.getPassword());
        return connectionProxy.getProxyConnection();
    }

    private ConnectionProxy getConnectionProxy(String username, String password) throws SQLException {
		
        ConnectionProxy connectionProxy = null;

        while (connectionProxy == null) {
            //做一个线程同步
            synchronized (monitor) {
                //空闲连接池不为空直接获取
                if (!idleConnections.isEmpty()) {
                    connectionProxy = idleConnections.remove(0);
                } else {
                    //没有空闲连接，就需要创建新连接
                    if (activeConnections.size() < super.getMaxActiveConnections()) {
                        //如果当前激活连接数小于最大值，允许创建
                        connectionProxy = new ConnectionProxy(super.getConnection(), this);
                    }
                    //否则不能创建，需要等待 poolTimeWait
                }
                if (connectionProxy == null) {
                    try {
                        monitor.wait(super.getPoolTimeWait());
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                        //有异常退出循环
                        break;
                    }
                }
            }
        }
        if (connectionProxy != null) {
            activeConnections.add(connectionProxy);
        }
        return connectionProxy;
    }

    public void colseConnection(ConnectionProxy connectionProxy) {
        synchronized (monitor) {
            //把激活状态的连接变成空闲连接
            activeConnections.remove(connectionProxy);
            if (idleConnections.size() < super.getMaxIdleConnections()) {
                idleConnections.add(connectionProxy);
            }
            //通知一下，唤醒上面那个等待获取连接的线程
            monitor.notify();
        }
    }
}
```