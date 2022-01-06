

## 代理模式

[参考链接](https://blog.csdn.net/A1342772/article/details/91349142)

> 定义：代理模式给某一个对象提供一个代理对象，并由代理对象控制对原对象的引用。通俗的来讲代理模式就是我们生活中常见的中介。

举个例子来说明：假如说我现在想买一辆二手车，虽然我可以自己去找车源，做质量检测等一系列的车辆过户流程，但是这确实太浪费我得时间和精力了。我只是想买一辆车而已为什么我还要额外做这么多事呢？于是我就通过中介公司来买车，他们来给我找车源，帮我办理车辆过户流程，我只是负责选择自己喜欢的车，然后付钱就可以了。

> 为什么要用代理模式？

在某些情况下，一个客户类不想或者不能直接引用一个真实类，而代理类对象可以在客户类和委托对象之间起到中介的作用，其特征是代理类和真实类实现相同的接口。

我们还可以通过给代理类增加额外的功能来扩展真实类的功能，这样做我们只需要修改代理类而不需要再修改真实类，符合代码设计的开闭原则。例如我们想给项目加入缓存、日志这些功能。

#### 静态代理

第一步：创建服务类接口

```java
public interface BuyHouse {
    void buyHosue();
}
```

第二步：实现服务接口

```java
public class BuyHouseImpl implements BuyHouse {
    @Override
    public void buyHosue() {
        System.out.println("我要买房");
    }
}
```

第三步：创建代理类

```java
public class BuyHouseProxy implements BuyHouse {
    private BuyHouse buyHouse;
    public BuyHouseProxy(BuyHouse buyHouse) {
        this.buyHouse = buyHouse;
    }
    @Override
    public void buyHosue() {
        System.out.println("买房前准备");
        buyHouse.buyHosue();
        System.out.println("买房后装修");
    }
}
```

> 缺点： **代理对象与目标对象要实现相同的接口，我们得为每一个服务都得创建代理类，工作量太大**，不易管理。同时接口一旦发生改变，代理类也得相应修改。 

#### 动态代理

> 动态代理有以下特点:
> 
> 1.代理对象,不需要实现接口
>
> 2.代理对象的生成,是利用JDK的API,动态的在内存中构建代理对象(需要我们指定创建代理对象/目标对象实现的接口的类型)
>
> 代理类不用再实现接口了。但是，要求被代理对象必须有接口。

```
Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)生成一个代理对象

参数1:ClassLoader loader 代理对象的类加载器 一般使用被代理对象的类加载器

参数2:Class<?>[] interfaces 代理对象的要实现的接口 一般使用的被代理对象实现的接口

参数3:InvocationHandler h (接口)执行处理类
```

```
InvocationHandler中的invoke(Object proxy, Method method, Object[] args)方法：调用代理类的任何方法，此方法都会执行

参数1:代理对象(慎用)

参数2:当前执行的方法

参数3:当前执行的方法运行时传递过来的参数
```

```java
public interface Girl {

    void date();

    void watchMovie();
}
```

```java
public class Wangmeili implements Girl{
    @Override
    public void date() {
        System.out.println("王美丽说：跟你约会好开心啊");
    }

    @Override
    public void watchMovie() {
        System.out.println("王美丽说：这个电影我不喜欢看");
    }
}
```

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class WangmeiliProxy implements InvocationHandler {
    //真实对象引入进来
    private Girl girl;

    public WangmeiliProxy(Girl girl) {
        this.girl = girl;
    }

    /**
     * 调用代理类的任何方法，此方法都会执行
     * @param proxy 代理对象(慎用)
     * @param method 当前执行的方法
     * @param args 当前执行的方法运行时传递过来的参数
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        doSomeThingBefore();
        //反射机制
        Object ret = method.invoke(girl, args);
        doSomeThingEnd();
        return ret;
    }

    private void doSomeThingBefore() {
        System.out.println("王美丽父亲说：我需要先调查他的背景");
    }

    private void doSomeThingEnd() {
        System.out.println("王美丽母亲说：他有没有对你动手动脚啊");
    }


    public Object getProxyInstance() {
        /**
         * 生成一个代理对象
         * 参数1:ClassLoader loader 代理对象的类加载器 一般使用被代理对象的类加载器
         * 参数2:Class<?>[] interfaces 代理对象的要实现的接口 一般使用的被代理对象实现的接口
         * 参数3:InvocationHandler h (接口)执行处理类
         */
        return Proxy.newProxyInstance(girl.getClass().getClassLoader(), girl.getClass().getInterfaces(), this);
    }
```

```java
public class XiaomingTest {
    public static void main(String[] args) {
        //隔壁有个女孩叫王美丽
        Girl girl = new Wangmeili();
        //她有个庞大的家庭，想跟她约会必须征得她家里人同意
        WangmeiliProxy family = new WangmeiliProxy(girl);
        //有一次我去约王美丽，碰到了她妈妈，我征得了她妈妈的同意
        Girl mother = (Girl)family.getProxyInstance();
        //通过她妈妈这个代理才能与王美丽约会
        mother.date();
        System.out.println("----------------------");
        //通过她妈妈这个代理才能与王美丽看电影
        mother.watchMovie();
    }
}
```

> **动态代理总结：**虽然相对于静态代理，动态代理大大减少了我们的开发任务，同时减少了对业务接口的依赖，降低了耦合度。但是还是有一点点小小的遗憾之处，那就是它始终无法摆脱仅支持interface代理的桎梏（我们要使用被代理的对象的接口），因为它的设计注定了这个遗憾。