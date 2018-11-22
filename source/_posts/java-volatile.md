---
cover: https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/jamstack-migration-netlify-4311d34cce03bd691fdb919e3f4a1dbf-a3725.jpg
date: 2018/11/22 20:46:25
title: 深入浅出java中volatile(非原创)
categories:
    - Java
tags:
    - java
---
> [原文](https://www.jianshu.com/p/e6a71c682ef1)

### 引言

这几天看了几篇关于java的volatile关键字的文章，今天就想总结一下关于volatile的相关知识巩固一下，虽然工作中很少能遇到volatile关键字，但是volatile也是java中很重要的知识点。

### volatile的官方定义

java语言规范第三版中对volatile的定义如下：java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致的更新，线程应确保通过排他锁单独获得这个变量。java语言提供了volatile，在某些情况下比锁更加方便。如果一个字段被声明为volatile，java线程内存模型确保所有线程看到这个变量的值是一致的。

### volatile的特性

从上面的官方定义我们可以看出 volatile实现了内存可见性
 内存可见性：简单的讲也就是说一个线程对 声明了volatile的变量进行修改，java会保证其他线程也能看见修改，保证内存中的变量是最新的（除volatile外 java中还有synchronized 和 final 能实现 可见性 这里不做赘述  后面再总结这个两个）
 原子性： volatile 能保证单个volatile的操作是原子性的 但不能保证 形如 i++这样的操作是原子性的 （下面会给出代码）

### volatile的使用

```
pubilc volatile static int i =0;
```

只要在变量前声明即可

### volatile原子性

volatile 无法保证复合性操作的原子性
 我们可以通过代码实验来证明这一点

```
pubilc class AtomicTest {
 volatile    static   int i =0;

    @Override
    public void run() {
        for(int k = 0 ;k<1000;k++){
            i++;
        }
    }

    public static void main(String[] args)throws InterruptedException{
        Thread[] threads = new Thread[10];
        for (int c = 0;c<10;c++){
            threads[c]=new Thread(new AtomicTest());
            threads[c].start();
        }
        for(int c =0;c<10;c++){
            threads[c].join();
        }
        System.out.println(i);
    }
}
```

如上， 执行上述代码你会发现每次输出的值都会小于我们所期待的最终值 10000 ，  也就 i++并不是原子性的。
 所以我们并不建议 volatile使用在这种场景下，我们可以用volatile ：检查某个标记以来判断是否进行下一步操作

```
pubilc class AtomicTest {
  volatile  boolean flag = false;
   pubilc void writer(){ 
    flag = true;
    }

   pubilc void reader(){
   if(flag){ 
      ........
     }
   }
}
```

### volatile的内存可见性

volatile的内存可见性是通过java内存模型对volatile定义的特殊规则定义的。在jvm虚拟机的内存模型中 分为本地内存和主内存，每一个线程都有自己的本地内存，并且共享同一个主内存

```
当写一个volatile变量时， java内存模型会把该变量从线程的本地变量刷新到主内存
当读一个volatile变量时，java内存模型会去主内存取该变量，然后将本地内存中的值改变
```

[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/4385259-2c36332ffe88609e.png)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/4385259-2c36332ffe88609e.png)

2.png


[![img](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/4385259-bc40b98181cd7b39.png)](https://wx-10045722.cos.ap-shanghai.myqcloud.com/blog-srping-boot-1/4385259-bc40b98181cd7b39.png)

3.png

### java内存模型中的volatile 、

java内存模型为了保证volatile 的内存可见性 对volatile还有第二条语义：禁止指令重排序优化（关于重排序后面我也会整理我的心得分享给大家，简单的讲就是java会对没有数据依赖的操作进行指令重排 已达到提升性能的目的）。
 那么为什么要禁止指令重排序呢？下面我们通过一个简单的代码来演示

```
Map mapTest;
char[] configuration;
volatile boolean init =false;
//假设a线程执行writer
public void writer(){
mapTest =new HashMap;
configuration = new char[10];//读之前的准工作
init =true;
}
//假设b线程执行该代码
pubilc void reader(){
if(init ){
  .......
  }
}
```

很简单的代码，如果init变量没有被定义为volatile的，那么 init=true 这段代码可能由于指令重排序的优化，导致其被提前执行，这样会导致配置b中使用a线程中的配置信息是出错，
 volatile变量的赋值的汇编代码是这样的

```
java代码 ： instance =new singleton（）
汇编代码：  0x01a3de1d: movb $0x0,0x1104800(%esi);
                     0x01a3de24: lock addl $0x0,(%esp);
```

其中 lock addl $0x0,(%esp) 就相当于一个内存屏障  对于volatile变量来说 读写会具有不同的内存屏障.
 具体会根据以下规则使用不同的内存屏障

> 当第二个操作是volatile写时，不管第一个操作是什么，都不>>能重排序。这个规则确保volatile写之前的操作不会被编译器重排序到volatile写之后。
>  当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序。这个规则确保volatile读之后的操作不会被编译器重排序到volatile读之前。
>  当第一个操作是volatile写，第二个操作是volatile读时，不能重排序。

### 结语

引用java并发编程实战中的一段话结束这篇文章
 仅当volatile 变量能简化代码的实习以及对同步策略的验证时，才应该使用它们，如果在验证正确性时需要对可见性进行复杂的判断，那么就不要使用volatile变量。volatile变量的正确使用方式包括：确保它们自身状态的可见性，确保它们所引用对象的状态的可见性，以及标示一些重要的程序生命周期时间的发生（例如，初始化和关闭）。