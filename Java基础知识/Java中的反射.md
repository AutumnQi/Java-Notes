# Java中的反射及应用

Java通过java.lang.reflect.*的`反射机制`提供了以下能力来动态操控Java的代码

- 在运行时分析类的能力。
- 在运行时查看对象， 例如， 编写一个 toString 方法供所有类使用。 
- 实现通用的数组操作代码。
- 利用 Method 对象， 这个对象很像C++中的函数指针。

## 1. Class, Field, Method, Constructor类

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

## 2. 动态代理

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
  	public void sayHello(){
      	System.out.println("hello world");
    }
}
```

#### 动态代理绑定和代理逻辑实现

通过实现java.lang.reflect.InvokationHandler接口来实现

```java
public class myProxyHelloWorld implements InvokationHandler {
  	//真实对象
  	private Object target = null;
  	//绑定代理对象和真实对象
  	public Object bind(Object target) {
      	this.target = target;
      	return Proxy.newProxyInstance(targrt.getClass().getClassLoader(),target.getClass().getInterfaces(),this);
      	//三个参数分别为要用到的类加载器，要将动态代理对象挂载的接口，实现代理逻辑的对象this
    }
  	//代理逻辑实现
  	@Override
  	public Object invoke(Object proxy,Method method,Object[] args){
      	//进入代理对象方法调用前的逻辑
      	System.out.println("doSomething Before");
      	Object obj = method.invoke(target,args);
      	//进入代理对象方法调用后的逻辑
      	System.out.println("doSomething After");
      	return obj;
    }
}
```

#### 应用方法

```java
public void testJDKProxy() {
  	myProxyHelloWorld proxy = new myProxyHelloWorld();
  	//通过HelloWorld这个接口来挂载代理类
  	HelloWorld hw = (HelloWorld)proxy.bind(new myHelloWorld());
  	hw.sayHello();
}
```

#### 源码分析

Step1 获取对应的proxyConstructor

```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h) {
        Objects.requireNonNull(h);
        final Class<?> caller = System.getSecurityManager() == null
                                    ? null
                                    : Reflection.getCallerClass();
        Constructor<?> cons = getProxyConstructor(caller, loader, interfaces);
        return newProxyInstance(caller, cons, h);
 }
```

Step2 通过proxyConstructor构建新的实例对象作为代理对象返回

```java
private static Object newProxyInstance(Class<?> caller, // null if no SecurityManager
                                           Constructor<?> cons,
                                           InvocationHandler h) {
        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (caller != null) {
                checkNewProxyPermission(caller, cons.getDeclaringClass());
            }
            return cons.newInstance(new Object[]{h});
        } catch ...
```

#TODO：再深一层的代码看不懂了0.0，先到此为止....

### CGLIB动态代理

#### 动态绑定和代理逻辑实现

JDK动态代理必须要有接口才能实现，而第三方技术CGLIB则只需要一个抽象类就能实现动态代理。

```java
public class cglibProxyExample implements MethodInterpretor{
  	//类似JDK的bind方法，返回一个与真实对象绑定了的代理对象
  	public Object getProxy(Class cls){
      	Enhancer enhancer = new Enhancer();
      	enhancer.setSuperclass(cls);
      	enhancer.setCallback(this);
      	return enhancer.create();
    }
  	//代理逻辑实现
  	@Override
  	public Object intercept(Object proxy,Method method,Object[] args,MethodProxy methodProxy) throws Throwable {
      	//进入代理对象方法调用前的逻辑
      	System.out.println("doSomething Before");
      	Object obj = methodProxy.invokeSuper(proxy,args);
      	//进入代理对象方法调用后的逻辑
      	System.out.println("doSomething After");
      	return obj;
    }
}
```

#### #TODO 源码分析

Step1 ？？？

```java
public Object invokeSuper(Object obj, Object[] args) throws Throwable {
        try {
            this.init();
            MethodProxy.FastClassInfo fci = this.fastClassInfo;
            return fci.f2.invoke(fci.i2, obj, args);
        } catch (InvocationTargetException var4) {
            throw var4.getTargetException();
        }
    }
```

Step2 ？？？

```java
public Object invoke(Object obj, Object[] args) throws Throwable {
        try {
            this.init();
            MethodProxy.FastClassInfo fci = this.fastClassInfo;
            return fci.f1.invoke(fci.i1, obj, args);
        } catch (InvocationTargetException var4) {
            throw var4.getTargetException();
        } catch (IllegalArgumentException var5) {
            if (this.fastClassInfo.i1 < 0) {
                throw new IllegalArgumentException("Protected method: " + this.sig1);
            } else {
                throw var5;
            }
        }
    }
```

## 3.  拦截器

#### 应用方法

用一个拦截器来对JDK动态代理进行包装，开发者只需要编写拦截器，设计者来完成复杂的动态代理逻辑的编写。

```java
public interface Interceptor {
  	public boolean before(Object proxy, Object target, Method method, Object[] args);  	
  	public boolean around(Object proxy, Object target, Method method, Object[] args);
  	public boolean after(Object proxy, Object target, Method method, Object[] args);
}
```

开发者只需要在实现类中完成逻辑即可

```java
public class myInterceptor implements Interceptor {
  	@Override
		public boolean before(Object proxy, Object target, Method method, Object[] args){
      	doSomething....
        return true;
    }
  	....
}
```

#### 实现原理

具体下图的拦截器实现逻辑由设计者来通过动态代理完成

```java
public class InterceptorJDKProxy implements InvokationHandler {
  	private Object target = null;
  	private String interceptorClass = null;
   
  	public InterceptorJDKProxy(Object target, String interceptorClass) {
      	this.target = target;
      	this.interceptorClass = interceptorClass;
    }
  	//绑定真实类和代理类
  	public static Object bind(Object target, String interceptorClass){
      return Proxy.newProxyInstance(target.getClass().getClassLoader(),target.getClass().getInterfaces(),new InterceptorJDKProxy(target, interceptorClass));
    }
  	//代理逻辑实现
  	@Override
  	public Object invoke(Object proxy,Method method,Object[] args) throws Throwable {
      	if(interceptorClass == null){
          	method.invoke(target,args);
        }
      	Object obj = null;
      	Interceptor interceptor = (Interceptor)Class.forName(interceptorClass).newInstance();
      	if(interceptor.before(proxy,target,method,args)){
          	result = method.invoke(target,args);
        }
      	else{
          	interceptor.around(proxy,target,method,args);
        }
      	interceptor.after(proxy,target,method,args);
      	return obj;
    }
}
```

#### 操作流程

- 在bind方法中用JDK动态绑定真实对象
- 如果没有设置拦截器，则直接反射真实对象的方法
- 若存在拦截器则通过反射生成拦截器对象
- 调用拦截器对象的before方法，若返回true，则直接反射真实对象的方法，否则调用around方法
- 调用after方法

<img src="/Users/inlab/Library/Application Support/typora-user-images/image-20201019190913810.png" alt="image-20201019190913810" style="zoom:50%;" />

#### 使用方法

```java
public static void main(String[] args){
  	HelloWorld proxy = (HelloWorld)MyInterceptor.bind(new myHelloWorld(),"myInterceptor");
  	proxy.sayHello();
}
```

#### 责任链模式

当拦截器存在多层嵌套时，这样的设计模式被称为责任链模式

- before**方法按照从最后一个拦截器到第一个拦截器的顺序运行**
- after**方法按照第一个拦截器到最后一个拦截器的顺序运行**















