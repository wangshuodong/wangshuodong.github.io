## 多数据源配置
1. 如果要开启多数据源在启动类`WangsdTemplateApplication`中增加
```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
```
2. 在`DataSourceType`类添加数据源枚举
3. 在`DruidConfig`配置读取数据源
```java
@Bean
@ConfigurationProperties("spring.datasource.druid.slave")
@ConditionalOnProperty(prefix = "spring.datasource.druid.slave", name = "enabled", havingValue = "true")
public DataSource slaveDataSource(DruidProperties druidProperties) {
    DruidDataSource dataSource = DruidDataSourceBuilder.create().build();
    return druidProperties.dataSource(dataSource);
}
```
4. 在`DruidConfig`类`dataSource`方法添加数据源
```java
setDataSource(targetDataSources, DataSourceType.SLAVE.name(), "slaveDataSource");
```
5. 在需要使用多数据源方法或类上添加`@DataSource`注解，其中`value`用来表示数据源
```java
@DataSource(value = DataSourceType.SLAVE)
public List<SysUser> selectUserList(SysUser user) {
	return userMapper.selectUserList(user);
}
```
```java
@Service
@DataSource(value = DataSourceType.SLAVE)
public class SysUserServiceImpl
```
6. **手动切换数据源**

    在需要切换数据源的方法中使用`DynamicDataSourceContextHolder`类实现手动切换，使用方法如下：
```java
public List<SysUser> selectUserList(SysUser user) {
	DynamicDataSourceContextHolder.setDataSourceType(DataSourceType.SLAVE.name());
	List<SysUser> userList = userMapper.selectUserList(user);
	DynamicDataSourceContextHolder.clearDataSourceType();
	return userList;
}
```
逻辑实现代码 `com.ruoyi.framework.aspectj.DataSourceAspect`

    `注意：目前配置了一个从库，默认关闭状态。如果不需要多数据源不用做任何配置。 另外可新增多个从库。支持不同数据源（Mysql、Oracle、SQLServer）`

!> 如果有`Service`方法内多个注解无效的情况使用内部方法调用`SpringUtils.getAopProxy(this).xxxxxx(xxxx)`;