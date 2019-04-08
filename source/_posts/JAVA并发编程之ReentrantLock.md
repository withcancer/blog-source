---
title: JAVA并发编程之ReetrantLock
date: 2016-8-3
categories:
- 后端
- JAVA
tags:
- 后端
- JAVA
- 并发
---

``ReentrantLock``类似于``synchronized``，有着相同的语义，可以实现重入锁的特性。但是相对于``synchronized``，``ReentrantLock``具有以下几个特性：
- ``ReenTrantLock``可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。
- ``ReenTrantLock``提供了一个``Condition``（条件）类，用来实现唤醒需要唤醒的线程们。
- ``ReenTrantLock``可以通过通过``lock.lockInterruptibly()``来实现锁中断机制。
- ``ReenTrantLockd``的等待可中断，持有锁的线程不释放的时候，正在等待的线程可以放弃等待，通过``tryLock``来实现。
- ``synchronized``在jvm层面实现，而``ReenTrantLock``通过代码实现，需要手动释放锁。
- 竞争激烈的情况下，``synchronized``的性能要弱于``ReenTrantLock``。
<!-- more -->
## 一个普通使用例子：
``` java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class Shit implements Runnable {
	
	final static ReentrantLock lock = new ReentrantLock();
	 final static Condition condition=lock.newCondition();
	static int count = 0;
	public static void main(String[] args) throws InterruptedException {
		Shit shit = new Shit();
		Thread t1 = new Thread(shit, "T1");
		t1.start();
		Thread.sleep(200); // 不等待会报错
		signal();
	}
	
	@Override
	public void run() {
		
		try {
			if(lock.tryLock()) {
				condition.await();
				for(int i=0; i<5;i++) {
					System.out.println(Thread.currentThread().getName() + ":" + (count++));
				}
			}else {
				System.out.println("Lock failed!");
			}
					
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally {
			lock.unlock();
		}
	}
	
	public static void signal() {
        try
        {
            if(lock.tryLock()) {
                condition.signal(); // 必须先lock再signal
            }else {
            	System.out.println("Lock failed!");
            }
        }
        finally
        {
            lock.unlock();
        }
	}
}


```

## 公平锁与非公平锁
``` java
public class Shit implements Runnable {

	final static ReentrantLock lock = new ReentrantLock(true);

	public static void main(String[] args) throws InterruptedException {
		Shit shit = new Shit();
		Thread t1 = new Thread(shit, "T1");
		Thread t2 = new Thread(shit, "T2");
		t1.start();
		t2.start();
	}

	@Override
	public void run() {
		while (true) {
			try {
				lock.lock();
				System.out.println(Thread.currentThread().getName() + ": get the lock");
			} finally {
				lock.unlock();
			}
		}
	}
}

/* 输出

T1: get the lock
T2: get the lock
T1: get the lock
T2: get the lock
......
*/

```
公平锁的前提下，两个线程会交替获得锁，当``ReentrantLock``没有设为``true``时，则是非公平锁，不会出现这种现象。同样地，``synchronized``也不会出现公平锁的交替获得锁的现象，而是随机的。
``` java
	@Override
	public void run() {
		while (true) {
			synchronized(this) {
				try {
					lock.lock();
					System.out.println(Thread.currentThread().getName() + ": get the lock");
				} finally {
					lock.unlock();
				}
			}
		}
	}
```
## 可中断锁
``` java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Shit {

	public static void test3() throws Exception {
		final Lock lock = new ReentrantLock();
		lock.lock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					lock.lock();
					// lock.lockInterruptibly();
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
					System.out.println(Thread.currentThread().getName() + " interrupted.");
				}
			}
		});
		t1.setName("T1");
		t1.start();
		Thread.sleep(1000);
		t1.interrupt();
	}

	public static void main(String[] args) throws Exception {
		test3();
	}
}
```
即使调用了``interrupt``，子线程仍然阻塞于获取锁。使用``lockInterruptibly``后，则会优先响应``interrupt``请求。

