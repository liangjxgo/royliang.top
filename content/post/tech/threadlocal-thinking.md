---
title: "一个关于ThreadLocal没有结果的思考"
date: 2018-07-05
tags: ["java", "threadlocal", "多线程"]
categories: ["tech"]
description: "想不到我的第一篇博文是分析ThreadLocal，当初遇见它的时候就感觉非常有缘，如今它有幸被我翻牌，那我们就来好好撩撩它吧！"
author: "liangjx"
---

> 想不到我的第一篇博文是分析ThreadLocal，当初遇见它的时候就感觉非常有缘，如今它有幸被我翻牌，那我们就来好好撩撩它吧！

---

## 话不多说，我们进入正题

我先说明，这篇博文是假定读者对ThreaLocal有一定认识的。

接下来，我们来看一段ThreadLocal的源码

```java
public void set(T value) {
       //获取当前线程
        Thread t = Thread.currentThread();
        //获取当前线程的ThreadLocalMap，也就是Thread类的threadLocals变量，
        ThreadLocalMap map = getMap(t);
        //如果为不为空就赋值，如果为空就创建一个ThreadLocalMap
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

从这段代码可以看到，ThreadLocal实际上就是，一旦线程调用了被ThreaLocal修饰的变量，则首先会获取该线程的实例，接着用过这个实例，去获取一个map（实际上是使用这个线程的地址作为一个key）。然后这个Map就是存放ThreadLocal修饰的变量的一个地方，网上很多博文都注明说是存放这个变量的一个拷贝，但是注意一下这段代码，传入的参数（T value），如果这里的T是一个引用类型，那么这个map里存放的只是这个引用类型的地址，一旦多个线程获取到这个T，同时操作，依然是会造成线程的不安全。下面这段代码可以很好的佐证我这个观点

```java
/** 创建一个ThradLocal实例 */
	private static ThreadLocal<StudentInfo> threadLocal = new ThreadLocal<StudentInfo>();

	public static void main(String[] args) {
		StudentInfo info = new StudentInfo("sdew23", "张三", "男");

		// 为主线程保存一个副本StudentInfo对象
		threadLocal.set(info);

		// 开启子线程
		new Thread(new Runnable() {
			@Override
			public void run() {
				threadLocal.set(info);
				StudentInfo infos = threadLocal.get();
				// 获取到子线程的变量数据然后修改一个属性
				infos.name = "哈哈";
				System.out.println(Thread.currentThread().getName() + "-->"
						+ infos);
			}
		}).start();
		// 打印主线程的变量
		System.out.println(Thread.currentThread().getName() + "-->"
				+ threadLocal.get());

	}
```

出现了3种结果：

1. `main-->StudentInfo{id='sdew23', name='张三', sex='男'}` / `Thread-0-->StudentInfo{id='sdew23', name='哈哈', sex='男'}`
2. `main-->StudentInfo{id='sdew23', name='哈哈', sex='男'}` / `Thread-0-->StudentInfo{id='sdew23', name='哈哈', sex='男'}`
3. `Thread-0-->StudentInfo{id='sdew23', name='哈哈', sex='男'}` / `main-->StudentInfo{id='sdew23', name='哈哈', sex='男'}`

**这段代码要用jdk1.8来编译，idea的话，language level也要1.8的。由于电脑的性能问题，很可能只会出现第一种情况，但是不要怀疑，多开几个线程就可以体现出来了。**

由于java多线程是抢占cpu执行权的策略，导致了这样的结果。
所以这个结果足以说明，ThreadLocal并不能使多个线程同时操作同一个共享变量达到线程安全。

这个博文是[另一个例子](https://blog.csdn.net/shiziaishuijiao/article/details/40153713/)，也是让人受益匪浅的，有需要的话，我也可以帮忙分析一下，其中还是有不少细节

---

查阅了一下午资料，多数博文都是认为，ThreadLocal是从堆中拷贝的一个副本保存在线程的工作栈中，可是通过我们刚才的实验来看，如果拷贝的这个是引用变量，那么即使是在本地线程中对这个变量进行了操作，那么也会影响本身的那个变量，因为线程本地栈的副本只是将引用地址，指向了本身变量。那么ThreadLocal这个修饰符还有什么作用呢？

一部分博文指出，ThreadLocal的使用场景应该是

- 初始化开销大的变量，类似connection之类的
- 需要频繁使用的变量
- 多线程操作，不会相互影响的变量，这里的操作应该是只读吧。

由于工作学习中很少用到ThreadLocal，只能粗略地谈论一下，本来以为是对相同输入，多条线程进行不同的操作，担心这个操作线程之间相互影响，因此用上ThreadLocal，可是通过源码，通过我们的实验，让我对ThreadLocal的使用场景更加迷茫了，一脸懵逼。
