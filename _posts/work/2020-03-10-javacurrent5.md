---
layout: post
title:  Java并发- 深入理解单例模式（五）
categories: Java并发编程
description: Java并发编程
keywords: Java, Java并发编程
---

模式是脱离语言的。

<a name="kVc2w"></a>
### 一、单例的模式由来

多线程要操作同一个对象，保证对象的唯一性。

如何解决？
> 实例化过程只实例化一次。


单例模式的四大原则 

> 1.构造方法私有化
> 2.以静态方法或者枚举返回实例
> 3.确保实例只有一个，尤其是多线程环境
> 4.确保反序列化时，不会重新构建对象


我们常见的单例模式有：

- 饿汉模式
- 懒汉模式
- 双重检索模式
- 静态内部类模式
- 枚举模式


<a name="q2iMn"></a>
### 二、单例模式的分类

<a name="KvDNa"></a>
#### 1.饿汉模式

```java

public class HungrySingleton {

    /**
     * 1.加载的时候就产生实例对象
     */
    private static HungrySingleton instance = new HungrySingleton();

    /**
     * 2.构造方法私有化
     */
    private HungrySingleton(){}

    /**
     * 3.提供返回实例对象的静态方法
     */

    public static HungrySingleton getInstance(){
        return instance;
    }

    public static void main(String[] args) {

        for (int i = 0; i < 50; i++) {

            new Thread(() ->{

                System.out.println(HungrySingleton.getInstance());
            }).start();
        }
    }
}
```

饿汉模式安全性：

- 在HungrySingleton类被类加载器加载的时候已经被实例化，所有只有这一次，以空间换时间，线程是安全的。
- 效率问题
  - 没有延迟加载，如果创建后不被使用，占内存，影响性能

<a name="PzEvW"></a>
#### 2.懒汉式

```java
public class LazySingleton {

    /**
     * 1.不进行初始化
     */
    private static LazySingleton instance = null;


    /**
     * 2.构造方法私有化
     */
    private LazySingleton(){};


    /**
     * 3.使用的时候进行初始化
     */
    public static LazySingleton getInstance(){
        if(Objects.isNull(instance)){
            instance = new LazySingleton();
        }
        return instance;
    }

    public static void main(String[] args) {
        /**
         * 模拟100个线程进行并发操作
         */
        for (int i = 0; i <10000; i++) {
            new Thread(() ->{
                System.out.println(LazySingleton.getInstance());
            }).start();
        }
    }
}

```

懒汉模式安全性：LazySingleton是在方法被调用后才创建对象，用时间换空间，在多线程环境下存在风险。

<a name="sKd8l"></a>
#### 3.双重检索懒汉模式（Double -Check -Lock）

```java
public class DCLSingleton {

    /**
     * 1.不进行初始化
     */
    private static DCLSingleton instance = null;

    /**
     * 2.构造方法私有化
     */
    private DCLSingleton(){}


    /**
     * 3.利用双重检索的方式进行初始化操作
     */
    public static DCLSingleton getInstance(){
        if(Objects.isNull(instance)){
            synchronized (DCLSingleton.class){
                if(Objects.isNull(instance)){
                    instance = new DCLSingleton();
                }
            }
        }
        return instance;
    }
}

```

上面的代码：在获取实例对象 getInstance()的方法中，我们首先判断instance是否为空，如果为空，则锁定DCLSingleton.class,并再次检查instance是否为空，如果还为空则创建DCLSingleton的一个实例。

我们假如有两个线程A,B同时调用getInstance（）方法，他们会同时发现instance ==null，于是同时对DCLSingleton.class加锁，此时JVM保证只有一个线程能够加锁成功（假如是线程A），另外一个线程会处于等待状态（假如是线程B）；线程A会创建一个DCLSingleton实例，之后释放锁，锁释放后，线程B被唤醒，线程B再次尝试加锁，此时是可以加锁成功的，加锁成功后，线程B检查Objects.isNull(instance)时会发现，已经创建过DCLSingleton实例了，所以线程B不会再创建一个DCLSingleton实例。

这看上去一切都很完美，无懈可击，但实际上这个getInstance（）方法并不完美，问题出在哪里呢，出在new 操作上，我们以为的new操作应该是：

1. 分配一块内存M
1. 在内存M上初始化DCLSingleton对象；
1. 然后M的地址赋值给instance变量

但可能经过JVM优化过后的执行顺序是这样的：<br />1.分配一个内存M；<br />2.将M的地址赋值instance变量<br />3.最后再内存上初始化DCLSingleton对象。

优化后会导致什么问题呢? 我们假设线程A先执行getInstance（）方法，当执行完指令2时恰好发生了线程切换，切换了线程上B上；如果此时线程B也执行getInstance（）方法，那么线程B 在执行第一个判断时会发现instance != null ，所以直接返回instance，而此时的instance是没有初始化过的，如果我们这个时候访问instance的成员变量就可能会触发空指针异常。

<br />![image.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4ubmxhcmsuY29tL3l1cXVlLzAvMjAyMC9wbmcvNDQwMjQ3LzE1ODQyNDQzNzAxMjItNDYzNWQ2MDctZTFlZC00ZWYzLThiYjktZjAzNDhiZWMzMTc0LnBuZw?x-oss-process=image/format,png#align=left&display=inline&height=640&name=image.png&originHeight=640&originWidth=1142&size=161634&status=done&style=none&width=1142)


<a name="o4cMJ"></a>
#### 4.双重检索（DCL）+ volatile
```java
public class DCLSingleton {

    /**
     * 1.不进行初始化
     */
    private volatile static DCLSingleton instance = null;

    /**
     * 2.构造方法私有化
     */
    private DCLSingleton(){}


    /**
     * 3.利用双重检索的方式进行初始化操作
     */
    public static DCLSingleton getInstance(){
        if(Objects.isNull(instance)){
            synchronized (DCLSingleton.class){
                if(Objects.isNull(instance)){
                    instance = new DCLSingleton();
                }
            }
        }
        return instance;
    }
}

```

volatile的作用：

1.保证可见性：
> 对共享变量的修改，其它线程马上能感知到 ,
> 这样volatile可以确保instance每次都会在主内存中读取，保证instance的一致性

2.保证有序性：
> 重排序：（编译阶段，指令优化阶段会进行重排序）
> as-if-serial： 重排序后在单线程不影响程序的执行结果，对多线程有影响
> volatile原则：
> volatile之前的代码不能调整到它的后面
> volatile之后的代码不能调整到它的前面
> 位置不变化


<a name="2UYmY"></a>
#### 5.静态内部类模式

```java

public class Singleton {
    /**
     * 1.构造方法私有化
     */
    private Singleton(){}

    /**
     * 2.定义静态内部类
     * 
     * 声明类的时候，成员变量中不声明实例变量，而放到内部静态类中
     */
    private static class SingletonHolder{
        private static Singleton instance = new Singleton();
    }

    /**
     * 3.返回实例对象
     */
    public static Singleton getInstance(){
        return SingletonHolder.instance;
    }
}

```

静态内部类的优点：
> 外部类加载时并不需要立即加载内部类，内部类不被加载则不去初始化instance，故而不占用内存，即当Singleton第一次被加载时，并不需要加载SingleHolder，只有当getInstance（）方法第一次被调用时，才会去初始化instance，第一次调用getInstance（）方法会导致JVM加载SingletonHolder类，这种方法不仅能保证线程安全，也能保证单例的唯一性，同时也延迟了到单例的实例化。


补充知识<br />当初始化一个类时，如果其父类还未进行初始化，会先触发其父类的初始化。

1. 遇到new、getstatic、setstatic或者invokestatic这4个字节码指令时，对应的java代码场景为：new一个关键字或者一个实例化对象时、读取或设置一个静态字段时(final修饰、已在编译期把结果放入常量池的除外)、调用一个类的静态方法时
1. 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没进行初始化，需要先调用其初始化方法进行初始化。
1. 当初始化一个类时，如果其父类还未进行初始化，会先触发其父类的初始化。
1. 当虚拟机启动时，用户需要指定一个要执行的主类(包含main()方法的类)，虚拟机会先初始化这个类。
1. 使用JDK 1.7等动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

<br />这5种情况被称为是类的主动引用，注意，这里《虚拟机规范》中使用的限定词是"有且仅有"，那么，除此之外的所有引用类都不会对类进行初始化，称为被动引用。静态内部类就属于被动引用的行列。

那么，是不是可以说静态内部类单例就是最完美的单例模式了呢？其实不然，静态内部类也有着一个致命的缺点，就是传参的问题，由于是静态内部类的形式去创建单例的，故外部无法传递参数进去，例如Context这种参数，所以，我们创建单例时，可以在静态内部类与DCL模式里自己斟酌。


<a name="VVzNi"></a>
####  6.枚举模式

```java
public class EnumSingleton {

    /**
     * 1.构造方法私有化
     */
    private EnumSingleton(){}

    private enum SingletonHolder{
        //创建一个枚举对象，该对象天生为单例
        INSTANCE;
        private EnumSingleton instance;

        SingletonHolder(){
            instance = new EnumSingleton();
        }

        public EnumSingleton getInstance(){
            return instance;
        }
    }

    /**
     * 对外暴露一个获取EnumSingleton对象的静态方法
     */
    public static EnumSingleton getInstance(){
        return SingletonHolder.INSTANCE.instance;
    }
    
}

```

