# Java中的反射

Java通过java.lang.reflect.*的`反射机制`提供了以下能力来动态操控Java的代码

- 在运行时分析类的能力。
- 在运行时查看对象， 例如， 编写一个 toString 方法供所有类使用。 
- 实现通用的数组操作代码。
- 利用 Method 对象， 这个对象很像中的函数指针。

## Class, Field, Method, Constructor类

在Java.lang.reflect包的基础上，Java虚拟机在类加载的同时维护一个对应的class对象，并包含了对应的Field, Method, Constructor类，分别来描述类的域、方法和构造器。任何一个该class的实例对象都可以通过`.getClass()`来获取其class对象。

在class对象的基础上，使用`getFields()`，`getMethods()`，`getConstructors()` 可以得到对应的public域、方法和构造器，**包括超类的公有成员**；

而使用`getDeclareFields()`，`getDeclareMethods()`，`getDeclareConstructors()` 则可以得到全部的域、方法和构造器，包括private和protected，但不包含超类

有了以上的反射机制，我们可以完成运行时类的分析，同时保证了可以通过配置来完成对象和方法的反射，在JavaBeans中得到了广泛的应用。

- 利用反射机制新建实例：

```java
//以下三种方式来获取class对象后，得到新的实例对象
1.Class.forName("packageName.className").newInstance()
2.int.Class.newInstance()
3.object.getClass().newInstance()
//若需要调用有参数的构造器则可采用如下的方式 
object.getClass().getConstructors(String.Class).newInstance("paramter")
//可以用==来比较两个对象是否同属一个类
if(obj1.getClass()==obj2.getClass()) ...
```

- 利用反射机制获取方法：

```java
//从class对象中获取需要的method对象
Method md=object.getClass().getMethods("methodName",arg1.Class,arg2.Class,...)
//调用方法，需要指定实例对象和输入的参数
md.invoke(instanceName,arg1,arg2,....)
```

## 动态代理

动态代理的逻辑在于：

完成代理需要分为两个步骤：

- 代理对象和真实对象建立代理关系
- 实现代理对象的处理逻辑

### JDK动态代理

JDK动态代理是java.lang.reflect.*包提供的方式，需要一个接口来生成代理对象。

```java
public interface HelloWorld {
  	public void helloWorld();
}

public class myHelloWorld implements HelloWorld {
  	public void helloWorld(){
      	System.out.println("hello world");
    }
}
```

通过实现java.lang.reflect.InvokationHandler接口来实现动态代理绑定和代理逻辑实现

```java
public class myProxyHelloWorld implements InvokationHandler {
  	//真实对象
  	private Object target = null;
  	//绑定代理对象和真实对象
  	public Object bind(Object target) {
      	
    }
}
```



