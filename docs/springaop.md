## 元注解

元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。
1. @Target
2. @Retention
3. @Documented
4. @Inherited

**@Target**

@Target说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。

**作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**

**取值(ElementType)有：**

1. CONSTRUCTOR:用于描述构造器
2. FIELD:用于描述域
3. LOCAL_VARIABLE:用于描述局部变量
4. METHOD:用于描述方法
5. PACKAGE:用于描述包
6. PARAMETER:用于描述参数
7. TYPE:用于描述类、接口(包括注解类型) 或enum声明

```java
@Target(ElementType.TYPE)
public @interface Table {
    /**
     * 数据表名称注解，默认值为类名称
     * @return
     */
    public String tableName() default "className";
}

@Target(ElementType.FIELD)
public @interface NoDBColumn {

}
注解Table 可以用于注解类、接口(包括注解类型) 或enum声明,而注解NoDBColumn仅可用于注解类的成员变量。
```

**@Retention：**

@Retention定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。

**作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）**

**取值（RetentionPoicy）有：**
1. SOURCE:在源文件中有效（即源文件保留）
2. CLASS:在class文件中有效（即class保留）
3. RUNTIME:在运行时有效（即运行时保留）

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
    public String name() default "fieldName";
    public String setFuncName() default "setField";
    public String getFuncName() default "getField"; 
    public boolean defaultDBValue() default false;
}
Column注解的的RetentionPolicy的属性值是RUTIME,这样注解处理器可以通过反射，获取到该注解的属性值，从而去做一些运行时的逻辑处理
```

**@Documented:**

@Documented用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

**@Inherited：**

@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。

　　注意：@Inherited annotation类型是被标注过的class的子类所继承。类并不从它所实现的接口继承annotation，方法并不从它所重载的方法继承annotation。

　　当@Inherited annotation类型标注的annotation的Retention是RetentionPolicy.RUNTIME，则反射API增强了这种继承性。如果我们使用java.lang.reflect去查询一个@Inherited annotation类型的annotation时，反射代码检查将展开工作：检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。

## 自定义注解

自定义注解使用场景
* 类属性自动赋值。
* 验证对象属性完整性。
* 代替配置文件功能，像spring基于注解的配置。
* 可以生成文档，像java代码注释中的@see,@param等

```java
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Target({ElementType.FIELD,ElementType.METHOD})
public @interface UserAnnotation {
    public String name() default "zhangsan";
    public String email() default "hello@example.com";
}
```

```java
public class User {
    @UserAnnotation(name = "zhang")
    private String name;

    @UserAnnotation(email = "zhang@example.com")
    private String email;

    @UserAnnotation(name = "sayHelloWorld")
    public String sayHello(){
        return "";
    }
}
```

```java
public class UserTest {
    /**
     *     getFields：只能访问public属性及继承到的public属性
     *     getDeclaredFields：只能访问到本类的属性，不能访问继承到的属性
     *
     *     getMethods：只能访问public方法及继承到的public方法
     *     getDeclaredMethods：只能访问到本类的方法，不能访问继承到的方法
     *
     *     getConstructors：只能访问public构造函数及继承到的public构造函数
     *     getDeclaredConstructors：只能访问到本类的构造函数，不能访问继承到的构造函数
     */
    public static void main(String[] args) {
        Method[] methods = User.class.getMethods();//获取User类的所有方法
//        Field[] fields = User.class.getFields();
        Field[] fields = User.class.getDeclaredFields();//获取User类的所有属性
        for (Field field : fields) {
            UserAnnotation annotation = field.getAnnotation(UserAnnotation.class);
            if (annotation != null) {
                System.out.println("property=" + annotation.name());
            }
        }
        for (Method method : methods) {
            UserAnnotation annotation = method.getAnnotation(UserAnnotation.class);
            if (annotation != null) {
                System.out.println("sayHello=" + annotation.name());
            }
        }
    }
}
```

## 面向切面编程

[例子](https://blog.csdn.net/fz13768884254/article/details/83538709)
>  @within和@annotation的区别
> @within 对象级别 
> @annotation 方法级别

* @Aspect 作用是把当前类标识为一个切面供容器读取
* @Pointcut 放在方法头上，定义一个可被别的方法引用的切入点表达式。
* @Before  在切点方法之前执行
* @After  在切点方法之后执行
* @AfterReturning 切点方法返回后执行
* @AfterThrowing 切点方法抛异常执行
* @Around 属于环绕增强，能控制切点执行前，执行后，，用这个注解后，程序抛异常，会影响@AfterThrowing这个注解

```java
@Target({ ElementType.PARAMETER, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Log
{
    /**
     * 模块 
     */
    public String title() default "";

    /**
     * 是否保存请求的参数
     */
    public boolean isSaveRequestData() default true;
}
```

```java
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.Arrays;

@Aspect
@Component
@Slf4j
public class AspectTest {

    @Pointcut("@annotation(com.wangsd.springAop.Log)")
    public void doPointCut() {

    }

    @Around(value = "doPointCut()")
    public Object methodAround(ProceedingJoinPoint point) throws Throwable {
        System.out.println("----methodAround----");
        System.out.println(point.getSignature());
        ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = requestAttributes.getRequest();
        log.debug("请求地址:" + request.getRequestURL().toString());
        log.debug("请求方式:" + request.getMethod());
        return point.proceed();
    }

    @Before(value = "doPointCut()")
    public void methodBefore(JoinPoint point) {
        MethodSignature signature = (MethodSignature) point.getSignature();
        System.out.println("----methodBefore----");
        System.out.println("目标方法为：" + point.getSignature().getDeclaringTypeName() +
                "." + point.getSignature().getName());
        System.out.println("参数为：" + Arrays.toString(point.getArgs()));
        System.out.println("被植入的目标对象为：" + point.getTarget());
    }

    @AfterReturning(value = "doPointCut()", returning = "result")
    public void methodAfterReturing(JoinPoint point, Object result) {
        System.out.println("----methodAfterReturing----");
    }

    @AfterThrowing(value = "doPointCut()", throwing = "ex")
    public void methodAfterThrowing(JoinPoint point, Throwable ex) {
        System.out.println("----methodAfterThrowing----");
    }
}
```
```java
@RestController
@RequestMapping("/aspect")
public class AspectController {

    @Log
    @RequestMapping("/test")
    public String test(String username){
//        int a = 1/0;
        return "成功";
    }
}
```