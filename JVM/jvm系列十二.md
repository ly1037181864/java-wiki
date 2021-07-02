String中的intern方法

一.intern方法的用途

　　在 JAVA 语言中有8中基本类型和一种比较特殊的类型String。这些类型为了使他们在运行过程中速度更快，更节省内存，都提供了一种常量池的概念。
  常量池就类似一个JAVA系统级别提供的缓存。8种基本类型的常量池都是系统协调的，String类型的常量池比较特殊。它的主要使用方法有两种：
  - 直接使用双引号声明出来的String对象会直接存储在常量池中。
  - 如果不是用双引号声明的String对象，可以使用String提供的intern方法。intern 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中

 

二.在JDK1.6和JDK1.7中的区别

```text
public static void main(String[] args) {
    String s1 = new String("1");//字节码层面两部1创建对象，2在常量池中创建该字符串常量对象
    s1.intern();
    String s2 = "1";
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");//约等于new String("11")，实际是jvm底层是调用String的append()和toString()方法，在toString()方法中构造的new String("11"),而toString()方法中构造的new String的构造器是char[]，因此并无直接用到"11"，所以最终只是创建了对象
    s3.intern();//将"11"添加到字符串常量池中，由于"11"在堆中已经有了，因此直接返回堆中的地址
    String s4 = "11";
    System.out.println(s3 == s4);
}
```
```text
@Override
    public String toString() {
        // Create a copy, don't share the array
        return new String(value, 0, count);
    }
```
![avar](../imags/jvm/jvm-69.png)    
![avar](../imags/jvm/jvm-71.png) 
　　在jdk1.6中返回false，false，jdk1.7是false，true

　　分析：

　　在JDK1.6下，StringPool在永久代，通过new关键字创建出来的有两个对象，一个在常量池，一个在堆中。

　　

　　上图可以容易的看出两个s1与s2， s3与s4的地址是不一样的，所以都返回false

　　在JDK1.7下，由于String大量的使用导致永久代经常发生OutOfMemoryError，所以将StringPool搬到heap中

　　

　　上图可以看出，为了不浪费heap的内存StringPool中的S3就直接引用了对象S3，所以S3与S4的地址是一致的

 

　　第二段代码

```text
public static void main(String[] args) {
    String s1 = new String("1");
    String s2 = "1";
    s1.intern();
    System.out.println(s1 == s2);

    String s3 = new String("1") + new String("1");
    String s4 = "11";
    s3.intern();
    System.out.println(s3 == s4);
}
```
![avar](../imags/jvm/jvm-70.png)    
![avar](../imags/jvm/jvm-72.png) 
　　分析第二段代码：

　　JDK1.6下，s1与s2，s3与s4的地址是不一致的，一个是堆中的地址，一个是常量池中的地址。

　　所以返回false，false。

　　JDK1.7下，s1与s2的地址不一致。s3创建好对象，但是s4在stringpool中创建常量，两个地址也是不一致。

　　所以返回false，false。

 

三.intern的使用技巧

　　适当的使用可以减少内存的消耗，每次new String都会产生一个新的对象，如果适当的使用String s=new String("1").intern();

能够适当的获取常量池的常量。

　　由于JDK1.6中String Pool大小限制在1009，底层是hash表加上链表组成，如果过度的使用，会导致链表过长从而导致速度变慢，所以

不是每个场景都适合使用intern方法（JDK1.7之后可以通过-XX:StringTableSize=99991设定hash表的大小）

 

四.总结　　　　　　　 

　　注意JDK1.6和JDK1.7中StringPool 的差异，以及大小的不同，合理的运用intern能给程序带来一定的好处，是系统更加健壮。

 

参考：https://tech.meituan.com/in_depth_understanding_string_intern.html