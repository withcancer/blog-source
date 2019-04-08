---
title: JAVA并发编程之ThreadLocal
date: 2016-8-5
categories:
- 后端
- JAVA
tags:
- 后端
- JAVA
- 并发
---

当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其他线程所对应的副本。从线程的角度看，目标变量就像是线程的本地变量。
<!-- more -->
下面是ThreadLocal的set方法源码，可以看出，set由以下几个步骤组成：
- 首先获取当前线程
- 从当前线程获取一个ThreadLocalMap的对象
- 如果上述ThreadLocalMap对象不为空，则设置值，否则创建这个ThreadLocalMap对象并设置值
所以，ThreadLocal的值是放入了当前线程的一个ThreadLocalMap实例中，所以只能在本线程中访问，其他线程无法访问。这就实现了“目标变量就像是线程的本地变量”这一特性。
```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

ThreadLocal的内存泄漏：

> ThreadLocalMap里面的key为ThreadLocal对象的弱引用，当一个线程调用ThreadLocal的set方法设置变量时候，当前线程的ThreadLocalMap里面就会存放一个记录，这个记录的key为ThreadLocal的引用，value则为设置的值。如果当前线程一直存在而没有调用ThreadLocal的remove方法，并且这时候其它地方还是有对ThreadLocal的引用，则当前线程的ThreadLocalMap变量里面会存在ThreadLocal变量的引用和value对象的引用是不会被释放的，这就会造成内存泄露的。但是考虑如果这个ThreadLocal变量没有了其他强依赖，而当前线程还存在的情况下，由于线程的ThreadLocalMap里面的key是弱依赖，则当前线程的ThreadLocalMap里面的ThreadLocal变量的弱引用会被在gc的时候回收，但是对应value还是会造成内存泄露，这时候ThreadLocalMap里面就会存在key为null但是value不为null的entry项。



用法实例：
``` java
public class Shit{
	private static ThreadLocal<Integer> seqNum = new ThreadLocal<Integer>() {
		public Integer initialValue() {
			return 0;
		}
	};

	public static void main(String [] args) {
		TestClient t1 = new TestClient();
		TestClient t2 = new TestClient();
		TestClient t3 = new TestClient();
		t1.start();
		t2.start();
		t3.start();
	}
	
	private static class TestClient extends Thread{
		public void run() {
			for(int i =0;i<3;i++) {
				seqNum.set(i);
				System.out.println(Thread.currentThread().getName() + ": seqNum = " + seqNum.get());
			}
		}
	}
}
/* 输出：
Thread-0: seqNum = 0
Thread-1: seqNum = 0
Thread-2: seqNum = 0
Thread-1: seqNum = 1
Thread-0: seqNum = 1
Thread-0: seqNum = 2
Thread-1: seqNum = 2
Thread-2: seqNum = 1
Thread-2: seqNum = 2
*/
```