---
layout: post                  
title: "字符串"             
date: 2018-09-18               
tag:  Java基础
---

## String StringBuffer StringBuilder比较

### 运行速度

***StringBuilder>StringBuffer>String***

String为字符串常量，而StringBuilder和StringBuffer均为字符串变量，即String对象一旦创建之后该对象是不可更改的，但后两者的对象是变量，是可以更改的

    String str="abc";
    System.out.println(str);
    str=str+"de";
    System.out.println(str);

第二行输出abc第四行输出abcde在JVM中对代码的处理为创建一个String对象str并把值"abc"赋给str在第三行中JVM又创建了一个新的str对象，把原来的str的值+"de"赋值给新的str，原来的str会被JVM的垃圾回收机制GC掉，所以str实际上没有被修改，也就是前面说的String类对象一旦创建无法修改。 **Java中对String对象进行的操作实际上是一个不断创建新的对象并且将旧的对象回收的一个过程**
StringBuffer与StringBuilder的对象是变量，对变量进行操作就是直接对该对象进行更改，而不进行创建和回收的过程，所以操作速度比String快的多

### 线程安全

在线程安全上，StringBuilder是线程不安全的，而StringBuffer是线程安全的，如果一个StringBuffer对象在字符串缓冲区被多个线程使用时，StringBuffer中很多方法可以带有synchronized关键字，所以可以保证线程是安全的，但StringBuilder的方法则没有该关键字，所以不能保证线程安全，有可能会出现一些错误的操作。所以如果要进行的操作是多线程的，那么就要使用StringBuffer，但是在单线程的情况下，还是建议使用速度比较快的StringBuilder。


### StringBuffer

而如果是使用 StringBuffer 类则结果就不一样了，每次结果都会对 StringBuffer 对象本身进行操作，而不是生成新的对象，再改变对象引用。 

### StringBuilder与StringBuffer

StringBuilder与StringBuffer有公共父类AbstractStringBuilder(抽象类)。

　　抽象类与接口的其中一个区别是：抽象类中可以定义一些子类的公共方法，子类只需要增加新的功能，不需要重复写已经存在的方法；而接口中只是对方法的申明和常量的定义。StringBuilder、StringBuffer的方法都会调用AbstractStringBuilder中的公共方法，如super.append(...)。只是StringBuffer会在方法上加synchronized关键字，进行同步。

### 总结

- String适用于少量的字符串操作的情况
- StringBuilder适用于单线程下在字符缓冲区进行大量操作的情况
- StringBuffer适用多线程下在字符缓冲区进行大量操作的情况


        String str="abc"+"de";
        StringBuffer sb=new StringBuffer("abc").append("de");
        //创建String对象的速度快于StringBufferJVM在创建String对象的时候执行String str="abcde";
        所以比StringBuffer快，但当操作不同对象的时候速度慢



