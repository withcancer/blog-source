---
title: JAVA并发编程之Synchronized
date: 2016-8-1
categories:
- 后端
- JAVA
tags:
- 后端
- JAVA
- 并发
---

JAVA中的``Synchronized``可以用来修饰以下几个对象：

- 代码块，作用范围是用大括号括起来的部分。
- 方法，作用范围是整个方法。
- 静态方法，作用范围是整个静态方法。
- 类，作用范围是``Synchronized``后面括号括起来的部分。

<!-- more -->

## ``Synchronized``作用于代码块
``` java
public class Shit implements Runnable {
	static int count = 0;
	public static void main(String[] args) throws InterruptedException {
		Shit shit = new Shit();
		Thread t1 = new Thread(shit, "T1");
		Thread t2 = new Thread(shit, "T2");
		t1.start();
		t2.start();
	}
	
	@Override
	public void run() {
		synchronized(this) {
			for(int i=0; i<5;i++) {
				System.out.println(Thread.currentThread().getName() + ":" + (count++));
			}
		}
	}
}
/* 
T1:0
T1:1
T1:2
T1:3
T1:4
T2:5
T2:6
T2:7
T2:8
T2:9
*/
```
当两个并发线程访问同一个synchronized代码块时，同一个时刻只能有一个线程得到执行，而另一个线程会被阻塞，直至这个线程执行完成。除了``this``外，也可以指定对象进行加锁，没有明确对象时，可以使用byte[0]来充当锁的对象。
## ``Synchronized``作用于方法
``` java
@Override
public void run() {
	countNumber();
}
	
private synchronized  void countNumber(){
	for(int i=0; i<5;i++) {
		System.out.println(Thread.currentThread().getName() + ":" + (count++));
	}
}
```
Synchronized修饰一个方法，就是在方法的前面加synchronized。和修饰一个代码块类似，只是作用范围不一样。前者是大括号内的代码，后者是整个方法。
## ``Synchronized``作用于静态方法
``` java
@Override
public void run() {
	countNumber();
}
	
private static synchronized  void countNumber(){
	for(int i=0; i<5;i++) {
		System.out.println(Thread.currentThread().getName() + ":" + (count++));
	}
}

public static void main(String[] args) throws InterruptedException {
	Thread t1 = new Thread(new Shit(), "T1");
	Thread t2 = new Thread(new Shit(), "T2");
	t1.start();
	t2.start();
}
```
静态方法属于类，两个线程相当于使用了同一把锁，因此可以线程同步。

## ``Synchronized``作用于类
``` java

	@Override
	public void run() {
		synchronized(Shit.class) {
			for(int i=0; i<5;i++) {
				System.out.println(Thread.currentThread().getName() + ":" + (count++));
			}
		}
	}
```
同上一个一致，都是对类加锁，两个线程相当于使用了同一把锁，因此可以线程同步。