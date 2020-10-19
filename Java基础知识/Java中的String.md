## String、StringBuider以及StringBuffer的区别和使用场景

### 为什么我们需要StringBuffer类和StringBuilder类

String是不可变的对象，因此在每次对String类型进行改变的时候，都会生成一个新的String对象，然后将指针指向新的String对象，所以经常改变内容的字符串最好不要用String，因为每次生成对象会降低性能，当内存中无引用对象多了GC就会开始工作，性能就会降低。

不要使用String类的"+"来进行频繁的拼接，因为性能是极差的，应该使用StringBuffer或StringBuilder类。

```java
String result = "";  
for (String s : hugeArray) {  
    result = result + s;  
}  
  
// 使用StringBuilder  
StringBuilder sb = new StringBuilder();  
for (String s : hugeArray) {  
    sb.append(s);  
}  

String result = sb.toString();  
```

### 区别

- StringBuilder与StringBuffer有公共父类AbstractStringBuilder(抽象类)。

- StringBuffer对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。

- StringBuilder并没有对方法进行加同步锁，所以是非线程安全的。

### 使用场景

- 当单线程操作大量数据时，建议使用StringBuilder，速度更快（10%~15%左右的性能提升），毕竟同步有性能开销。

- 多线程操作大量数据时，建议使用StringBuffer。可用于全局变量中。

相同情况下StirngBuilder虽然比StringBuffer获得10%~15%左右的性能提升，但却要冒多线程不安全的风险。而在现实的模块化编程中，负责某一模块的程序员不一定能清晰地判断该模块是否会放入多线程的环境中运行，因此除非确定系统的瓶颈是在StringBuffer上，并且确定你的模块不会运行在多线程模式下，才可以采用StringBuilder，否则还是用StringBuffer。

### 实现原理

我们知道使用StringBuffer和StringBuider无非就是为了提高字符串连接的效率，因为直接使用+进行字符串连接会创建多个String对象，造成一定的开销。

AbstractStringBuilder中采用一个char数组来保存字符串，char数组有一个初始大小（16），当append的字符串长度超过当前char数组容量时，则对char数组进行动态扩容，然后将当前char数组拷贝到新的位置，因为重新分配内存并拷贝的开销比较大，所以每次都是进行固定扩容2倍的方式（使用JNI方法System.arraycopy()）。

最后就是，为了获得更好的性能，在构造StirngBuffer或StirngBuilder时应尽可能指定它们的容量。当然，如果你操作的字符串长度（length）不超过16个字符就不用了，因为默认容量为16。不指定容量会显著降低性能。

```java
StringBuilder();//创建一个容量为16的StringBuilder对象（16个空元素）
StringBuilder(int initCapacity);//创建一个容量为initCapacity的StringBuilder对象
StringBuilder(CharSequence c);//创建一个包含c的StringBuilder对象，末尾附加16个空元素
StringBuilder(String s);//创建一个包含s的StringBuilder对象，末尾附加16个空元素
```

## String对象的创建和比较

- 基本数据类型之间使用"=="来比较其值
- 符合数据类型（对象）之间使用"=="来比较其内存地址，而使用.equal()方法来比较值

### 特殊情况的String

String由于其不可变的特性，直接赋值的情况下不会在堆中新建一个String对象，而是在类加载的解析阶段被加载到常量池中，可能指向的都是同一个对象，如下所示

```java
String a="abs";
String b="abs";
a==b 的结果为True
String c=new String("abs");
a==c 的结果为False
```

### 题外话Interger的比较

```java
Integer i1 = 100,i2 = 100;
i1==i2 的结果为True
Integer i3 = 1000,i4 = 1000;
i3==i4 的结果为False
```

查看Integer.java类，会发现有一个内部私有类，IntegerCache.java，它缓存了从-128到127之间的所有的整数对象。

所以例子中i1和i2指向了一个对象。因此100==100为true。

### JDK1.6 中 String的Intern方法

```java
String a="abs";
String b="abs";
a==b 的结果为True
String c=new String("abs");
c=c.intern();
a==c 的结果为True
```

intern方法的作用为：检查字符串池里是否存在"abc"这么一个字符串，如果存在，就返回池里的字符串；如果不存在，该方法会把"abc"添加到字符串池中，然后再返回它的引用

### JDK1.7 中 String的Intern方法

在JDK1.6之前，还会明确区分堆和方法区，而在JDK 1.7后，intern方法还是会先去查询常量池中是否有已经存在，如果存在，则返回常量池中的引用，这一点与之前没有区别，区别在于，如果在常量池找不到对应的字符串，则不会再将字符串拷贝到常量池，而只是在常量池中生成一个对原字符串的引用。简单的说，就是往常量池放的东西变了：原来在常量池中找不到时，复制一个副本放到常量池，1.7后则是将在堆上的地址引用复制到常量池。

```java
String str1 = new String("SEU") + new String("Calvin");
System.out.println(str1.intern() == str1);
System.out.println(str1 == "SEUCalvin");

两个输出都为True
```

str1.intern()发现常量池中不存在“SEUCalvin”，因此指向了str1。 "SEUCalvin"在常量池中创建时，也就直接指向了str1了。两个都返回true就理所当然啦。



## 参考链接

https://blog.csdn.net/tyyking/article/details/82496901