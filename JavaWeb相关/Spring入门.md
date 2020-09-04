# 什么是Spring

- 为解决企业级应用开发的复杂性而创建

- 创建Spring的目的就是用来**替代更加重量级的的企业级Java技术**

- **简化Java的开发**

- - 基于POJO轻量级和**最小侵入式开发**
  - 通过依赖注入和面向接口实现**松耦合**
  - **基于切面**和惯例进行声明式编程
  - 通过切面和模板**减少样板式代码 **

**侵入式概念**

- 对于EJB、Struts2等一些传统的框架，**通常是要实现特定的接口，继承特定的类才能增强功能**

- - **改变了java类的结构**

**非侵入式**

- 对于Hibernate、Spring等框架，**对现有的类结构没有影响，就能够增强JavaBean的功能**

# Ioc

> Inversion of control 控制反转，通过描述（XML或注解）从第三方处获取或生成特定的对象

传统的构架中，A对象中创建了一个B对象，则A对B有着控制权，Ioc的概念及将二者解耦，即把B对象的创建过程交给Spring容器，A对象向容器来申请B的使用，但B的创建和销毁由容器来统一管理。A不需要知道B对象的内部细节，只需要知道根据输入去获得对应的对象。

## Ioc容器的创建及注入

- `ClassPathXmlApplicationContext`---->`ApplicationContext`---->`BeanFactory`以上为三个接口的继承关系，一般使用第三层级的子接口来实现具体的Ioc容器
- 如在spring的`appContext.xml`文件中进行编辑，实现基于`ClassPathXmlApplicationContext`的容器

## Ioc容器的初始化

- 初始化的过程共三步

  - Resource定位
  - BeanDefinition的载入，根据配置获得对应的POJO(Plain Ordinary Java Object)即普通Java类，完成类的加载？
  - BeanDefinition的注册，将第二步中获取的POJO在容器中进行注册

  完成初始化后，每个容器中的Bean还未完成注入，并不能使用

## Ioc容器的注入

实现对象的实例化

#### 1.使用XMl文件来注入

- 在`appContext.xml`文件中完成Bean的注入方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
		//注入方法1：利用构造方法
    <bean class="org.javalearn.ioc.model.User" id="user">
        <constructor-arg name="id" value="1"/>
        <constructor-arg name="username" value="javalearner"/>
        <constructor-arg name="address" value="www.learner.prg"/>
    </bean>
		//注入方法2：利用属性set
    <bean class="org.javalearn.ioc.model.User" id="user2">
        <property name="id" value="2"/>
        <property name="username" value="javalearner"/>
        <property name="address" value="www.learner.org"/>
    </bean>
		//注入方法3（该方法会二次调用容器的初始化过程）：利用命名空间
    <bean class="org.javalearn.ioc.model.User" id="user3" p:id="3" p:username="users" p:address="user3 address"/>
  	//注入方法4:使用工厂方式注入，先注入工程类，再调用其实例方法
  	<bean class="org.javalearn.ioc.OkHttpStaticFactory" id="okHttpStaticFactory"/>
    <bean class="okhttp3.OkHttpClient" factory-bean="okHttpStaticFactory" factory-method="getInstance" id="okHttpClient"/>
  	//注入方法5:对象的注入
  <bean class="org.javalearn.ioc.model.User" id="user4">
        <property name="id" value="2"/>
        <property name="username" value="javalearner"/>
        <property name="address" value="www.learner.org"/>
				//使用ref来对其他对象进行引用以实现注入
        <property name="cat" ref="cat"/>
    		//对数组进行注入
        <property name="favorites">
            <list>
                <value>Baseball</value>
            </list>
        </property>
				//对对象数组进行注入
        <property name="cats">
            <array>
                <ref bean="cat"/>
                <bean class="org.javalearn.ioc.model.Cat" id="cat2">
                    <constructor-arg name="name" value="Henlen"/>
                </bean>
            </array>
        </property>
				//对map进行注入
        <property name="details">
            <map>
                <entry key="Age" value="18"/>
                <entry key="wealth" value="100000"/>
            </map>
        </property>
				//对property进行注入
        <property name="info">
            <props>
                <prop key="phone">12345678</prop>
            </props>
        </property>
    </bean>
</beans>
```

- 使用`ClassPathXmlApplicationContext`来获取容器

```java
public class Main {
    public static void main(String[] args) {
        //将对象交给容器，再向容器去要，由容器负责创建和销毁，但得到的instance都是同一个
        //TODO：xml内部的类都会执行一次无参数的构造方法？
       ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("appContext.xml");
        //使用bean的id来获取不同的bean对象
      	User u1=(User)ctx.getBean("user");
        User u2=ctx.getBean("user2",User.class);
        User u3=ctx.getBean("user3",User.class);
        System.out.println("u1 = " + u1);
        System.out.println("u2 = " + u2);
        System.out.println("u3 = " + u3);
    }
}
```

#### 2.使用Java代码实现Java配置类进行注入——称为注解

- 在JavaConfig.java中实现各个Bean的注入

```java
@Configuration//标志为JavaConfig类，取代XML文件
public class JavaConfig {
    @Bean
    SayHello sayHello(){//相当于id
        return new SayHello();//内部完成各种复杂的注入过程，返回注入完成的对象
    }
}
```

- 使用`AnnotationConfigApplicationContext`来获取容器

```java
public class JavaConfigTest {
    public static void main(String[] args) {
   			AnnotationConfigApplicationContext ctx = new 			
          														AnnotationConfigApplicationContext(JavaConfig.class);
        SayHello sayHello = ctx.getBean("sayHello",SayHello.class);
        sayHello.sayhello("John");
    }
}
```

#### 3.自动化扫描进行配置和注入

- 在需要被扫描的类中加上装饰器即可实现自动化的配置
  - 在service层上使用@Service
  - 在Dao层使用@Repository
  - 在Controller层使用@Controller
  - 在其他组件上使用@Component

```java
@Service("us")
public class UserService {
  	@Autowired//注入对象的方式
  	UserDao userdao;
    public List<String> getAllUsers(){
        ArrayList<String> all = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            all.add("User: "+i);
        }
        return  all;
    }
}
```
- 在JavaConfig.java中略作修改

```java
@Configuration//标志为JavaConfig类，取代XML文件
@ComponentScan("org.javalearn.ioc.service")//指定扫描该路径下的包以自动注册到容器中，默认的bean id为类名的小写，可以进一步指定要扫描的类
public class JavaConfig {
    @Bean
    SayHello sayHello(){//相当于id
        return new SayHello();//内部完成各种复杂的注入过程，返回注入完成的对象
    }
}
```
- 在xml文件中加入自动注入生成

```xml
<context:component-scan base-package="org.javalearn.ioc.service" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Service"/>
    </context:component-scan>
```

- 对象注入
  - @Autowired
    - 根据类去查找，单例
    - 可以配合@Qualified来实现根据变量名查找
  - @Resource
    - 根据名称去查找
  - @injection

- 几个问题
  1. 自动扫描得到的Bean名字默认为类名的小写，但也可以在@service后加上自定义的名称
     - #TODO 所以如何实现xml中同一个类注入的多个对象？
  2. 几种指定的扫描方式
     - 包的地址
     - 筛选方式
  3. xml文件中bean的`id`和`name`有什么区别
     - name="n1,n2,n3"则使用n1 n2 n3都可以获取到这个bean
     - id="n1,n2,n3"则相当于一个整体

#### 4. 条件注解 @Condition

- 详见ioc02项目
- 新建实现Spring中Condition接口的条件类
- 在Bean后添加一个@Condition来使用条件注解，输入为条件类的class属性（再次利用反射获取类对象）

#### 5.环境切换 @Profile 

- 在不同的profile下可以有不同的注入方式
- 在xml文件中体现在<beans profile="dev"><\beans>
- 在JavaConfig中体现在@Profile
  - 在容器初始化阶段不传入JavaConfig.class
  - 使用register来进行不同环节的注册
  - 再传入JavaConfig.class，最后refresh

#### 6.Bean的作用域

- 在xml文件中，可以指定scope来控制每一次获取的是否是同一个对象

  ```xml
  //scope="prototype"表示每次都会新建一个对象
  //scope="singleton"表示只有一个对象
  //默认为singleton
  <bean class="org.javalearn.ioc.model.User" id="user" scope="prototype">
          <constructor-arg name="id" value="1"/>
          <constructor-arg name="username" value="javalearner"/>
          <constructor-arg name="address" value="www.learner.prg"/>
      </bean>
  ```

- 在JavaConfig中设置@Scope("prototype")即可

# Aop

>  oop是面对切面编程

### 使用动态代理实现Aop

- 原始类

```java
public class MyCalculator1 implements Calculator{
    public int add(int a, int b) {
        return a+b;
    }
}
```

- 代理类

```java
public class MyCalculatorProxy {
    public static Object getInstance(final MyCalculator1 calculator){
        return Proxy.newProxyInstance(MyCalculatorProxy.class.getClassLoader(), calculator.getClass().getInterfaces(), new InvocationHandler() {
            /**
             * @param proxy 代理对象
             * @param method 代理方法
             * @param args 代理参数
             * @return
             * @throws Throwable
             */
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println(method.getName()+" begin!");
                Object invoke = method.invoke(calculator, args);
                System.out.println(method.getName()+" end!");
                return invoke;
            }
        });
    }
}
```

### AOP的五种通知

- 前置通知：在目标方法invoke前执行
- 后置通知
- 异常通知
- 返回通知
- 环绕通知

##### 实现方式

- 利用注解实现

生成一个切面，并在切面上实现附加的功能模块

```java
@Component
public class MyCalculator1 implements Calculator{
    @Action
    public int add(int a, int b) {
        return a+b;
    }

    public void minus(int a, int b) {
        System.out.println(a-b);
    }
}

@Component
@Aspect//表示这是一个切面
public class LogAspect {

    /**
     * 优化方式1：统一定义一组切点
     */
    @Pointcut(value = "@annotation(Action)")
    public void pointCut(){

    }

    /**
     * "* org.javalearn.aop.MyCalculator1.*(..))"
     * "返回类型 包.类.方法(参数类型，参数个数)."
     * 即指定了某些函数，定义了一些pointCut
     */
    @Pointcut(value = "execution(* org.javalearn.aop.MyCalculator1.*(..))")//相当于写了一个拦截的规则，不用在源代码上写@Actio注解
    public void pointCut1(){

    }

    /**
     * 前置通知
     * @param joinPoint 切面所在的位置
     */
    @Before(value = "pointCut()")//在标明Action的方法执行前来执行
    public void before(JoinPoint joinPoint){
        String name = joinPoint.getSignature().getName();
        System.out.println(name + " method begin!");
    }

    @After(value = "@annotation(Action)")
    public void after(JoinPoint joinPoint){
        String name = joinPoint.getSignature().getName();
        System.out.println(name + " method end!");
    }

    /**
     * 返回通知，使用方法的返回值
     * @param joinPoint
     * @param r 方法的返回对象，void方法的返回对象为null
     */
    @AfterReturning(value = "@annotation(Action)",returning = "r")//有对应类型的返回值才会调用
    public void returning(JoinPoint joinPoint,Integer r){
        String name = joinPoint.getSignature().getName();
        System.out.println(name+" 方法返回通知: "+r);
    }

    /**
     * 异常通知，使用异常
     * @param joinPoint
     * @param e 注意异常的类型，可以指定接受异常的类型
     */
    @AfterThrowing(value = "@annotation(Action)",throwing = "e")
    public void afterThrowing(JoinPoint joinPoint, Exception e){
        String name = joinPoint.getSignature().getName();
        System.out.println(name+" 异常返回通知: "+e.getMessage());
    }

    /**
     * 以上四种方法的集成，使用反射机制实现
     * @param pjp
     * @return
     */
    @Around(value = "@annotation(Action)")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        //类似动态代理的method invoke过程
        Object proceed = pjp.proceed();
        return proceed;
    }
}
```

- 利用规则实现

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
        <bean class="org.javalearn.aop.LogAspectXml" name="logAspectXml"/>
        <aop:config>
            <aop:pointcut id="pt1" expression="execution(* org.javalearn.aop.MyCalculator1.*(..)))"/>
            <aop:aspect ref="logAspectXml">
                <aop:before method="before" pointcut-ref="pt1"/>
                <aop:after method="after" pointcut-ref="pt1"/>
            </aop:aspect>
        </aop:config>
</beans>
```



























