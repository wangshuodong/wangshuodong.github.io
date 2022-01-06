## 国内解决github无法访问的问题
> [第一种办法](https://www.jianshu.com/p/8fcc2acd77a7)
> [第二种办法](https://www.cnblogs.com/lvchaoshun/p/14646135.html)
```shell
140.82.112.4      github.com
151.101.1.194    github.global.ssl.fastly.net
```

## 安装翻墙软件
> [教程地址](https://www.jamesdailylife.com/v2ray-vless-tcp-xtls)
```shell
wget -P /root -N --no-check-certificate "https://raw.githubusercontent.com/mack-a/v2ray-agent/master/install.sh" && chmod 700 /root/install.sh && /root/install.sh
```
## JPA生成表

> fastmybatis需要的注解
>
> ```java
> @Id
> @GeneratedValue(strategy = GenerationType.IDENTITY)
> private Long id;
> 
> @Transient
> private String noName;
> ```
> 
> 

> 依赖
>
> ```xml
> <!-- springDataJpa 生成数据库表-->
> <dependency>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-starter-data-jpa</artifactId>
> </dependency>
> ```
>
> 

> 配置
>
> ```yaml
> spring:
>   #jpa自动生成表
>   jpa:
>     show-sql: true
>     hibernate:
>       ddl-auto: update
> ```
>
> 

```java
/**
 * 有两个@Table，第一个@table是jpa自带的，第二个是hibernate的，必须结合使用才能生成表注释…
 * 经过测试，如果把jpa的@table删除，生成，是无反应的（无效的）
 */
@Data
@Entity
@Table(name = "test")
@org.hibernate.annotations.Table(appliesTo = "test", comment="测试表")
public class Test {
    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, columnDefinition = "varchar(50) default '' comment '姓名' ")
    private String name;

    @Column(nullable = false, columnDefinition = "int(2) comment '年龄' ")
    private Integer age;
}
```