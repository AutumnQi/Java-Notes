# Java中的注解

> 参考：《Java核心技术 卷2》

## 1. 什么是注解

注解是一种`标签`🏷️，可以在源代码中进行插入，随后可以调用***相应的工具***来对其进行处理。

这些工具可以在

1. Runtime中进行处理
2. 源代码的层次上进行处理
3. 编译器的层次上进行处理

！但是，注解不会改变虚拟机的编译方式，最后生成的虚拟机指令是一致的。

## 2. 注解的工作原理

注解本身并不会做任何事情，只是相当于在某个代码片段前加了一个标签，需要用对应的工具去读取这些标签来进行处理。每一个注解都必须通过一个**注解接口**进行定义，再实现一个具体的类用来进行实际的处理。

注解接口的示例如下：

```java
@Target(ElementType.Method)
@Retention(RetentionPolicy.Runtime)//以上为元注解，所有的注解都会用到
public @interface AnnotationName {
  //注解元素的格式如下
  type elementName() default value;
  ....
}

@AnnontationName(elementName1=value1,elementName2=value2,...)
public void doSomething(){...}
```

### 在Runtime处理注解的工作流程

首先需要实现一个Java工具类来处理具体的注解，在Runtime中按照以下的流程来工作：

![image-20201021152748733](/Users/inlab/Library/Application Support/typora-user-images/image-20201021152748733.png)

- 对某个对象使用`getDeclaredMethods()`方法，获取其所有方法
- 对每个方法调用`getAnnotation(XXAnnotation.class)`方法，获取某一类注解
- 构造一个实现了注解接口的[代理对象](https://blog.csdn.net/xiaowu_zhu/article/details/83019440)，在该对象中调用被注解的方法。

```java
public static void processAnnotation(Object obj){
  	try{
      	Class<?> cl = obj.getClass();
      	for(Method m: cl.getDeclaredMethods()){
          	//获取每个方法的某个指定注解
          	XXAnnotationName a = m.getAnnotation(XXAnnotationName.class);
          	if(a != null){
              	//获取注解中传入的参数
              	Field f = cl.getDeclaredField(a.source());
              	f.setAccessible(true);
              	//doSomething by Proxy Object;
              	.....
            }
        }
    }
}
```

> #问题：是否每个对象都需要判断是否有注解？如何确定改判断的注解类型？这里的例子是方法前的注解，那class等声明前的注解要如何处理？

因此注解是编译器计算来的，所以所有的传入注解的value必须是编译器常量！

- 基本类型
- String
- Class（具有一个可选的类型参数 Class<?>）
- enum
- 注解类型
- 由前述类型组成的数组

## 3.注解出现的位置

### 3.1 在声明前

包括：包、类、接口、方法、构造器、实例域、局部变量、参数变量、类型参数

声明注解提供了正在被声明的项的信息，如下断言userId非空

```java
public User getUser(@NonNull String userId)
```

### 3.2 在指定类型前

将注解放在引用类型元前，如下表示List中的所有字符串都不为空

```java
List<@NonNull String>
```

### 3.3 注解this

在输入的参数中对this有注解需求的，可以用Ponit this来指定this

```java
public boolean equals(@ReadOnly Point this, @ReadOnly Object other) {...}
```

## 4.标准注解

![标准注解1](../Images/标准注解1.png)

![标准注解2](../Images/标准注解2.png)

以上为Java SE中给出的标准注解。

## 5. 源码级注解处理

> 源码级的注解处理仅仅生成新的助手类源文件，并和原始的源文件一同被加载到虚拟机中

不同的注解处理器都已经被集成到了Java编译器中。在编译过程中，每个注解处理器会依次执行，得到其对应的注解片段后创建一个新的源文件。在这个源文件的基础上再次执行处理过程，直到不再产生任何新的源文件，编译所有的源文件。

具体的实现原理是基于AST的语言模型API，通过AST的节点TypeElement, VariableElement, ExecutableElement等接口的类的实例来获取需要的信息，具体的步骤如下：

- RoundEnvrionment通过调用以下的方法得到一个特定注解标注过的所有元素构成的集合

  ```java
  Set<? extends Element> getElementsAnnotatedWith(Class<? extends Annotation> a)
  ```
  
- 也可以通过下面的方法得到某一个TypeElement给定注解类的注解

  ```java
  A getAnnotation(Class<A> annotationType)
  A[] getAnnotationsByType(Class<A> annotationType)  
  ```

- 调用getEnclosedElements方法得到该TypeElement下所有的VariableElement, ExecutableElement进行相应的处理，即生成新的源代码并保存到助手类的源文件中。

注解处理器生成的助手类的源文件也可以是XML描述符、属性文件、Shell脚本、HTML文档等。

## 6. 虚拟机级注解处理

> 虚拟机级的注解处理直接在字节码生成阶段生效，即直接改变了原始文件的编译结果

实现原理如下：

- 加载类文件中的字节码
- 定位所有的方法
- 对于每个方法，检查是否有相关的注解
- 如果有，在方法开始的部分添加字节码

例如LogEntry注解可以实现每次调用该方法时在日志中打印信息

```java
@LogEntry(logger="global")
public int hashCode(){....}
```
为了保证效率，在执行过程中一般在类加载时对字节码进行修改

![image-20201021163651388](/Users/inlab/Library/Application Support/typora-user-images/image-20201021163651388.png)  

