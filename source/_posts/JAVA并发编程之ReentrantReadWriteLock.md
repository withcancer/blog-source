---
title: JAVA并发编程之ReentrantReadWriteLock
date: 2016-8-3
categories:
- 后端
- JAVA
tags:
- 后端
- JAVA
- 并发
---

读写分离锁可以有效地帮助减少竞争，由于读操作并不对数据完整性造成破坏，因此，读写锁允许多个线程同时进行读操作，使得多个线程间真正并行。但是读写操作间仍然是需要相互等待和持有锁的，这些关系可以用下图来表示：

   |读 |写 
---|---|---
读 | 非阻塞| 阻塞
写 | 阻塞  | 阻塞

下面用一个例子来对比``ReentrantReadWriteLock``和``ReentrantLock``。
<!-- more -->
``` java
package fuck;

import java.util.Random;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class Shit{
	
	private static Lock lock = new ReentrantLock();
	private static ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
	private static Lock rLock = rwLock.readLock();
	private static Lock wLock = rwLock.writeLock();
	private int value;
	
	public Object handleRead(Lock lock) throws InterruptedException{
		try {
			lock.lock();
			Thread.sleep(1000);
			System.out.println(Thread.currentThread().getName() + ": read value" + value);
			return value;
		}finally {
			lock.unlock();
		}
	}
	
	public void handleWrite(Lock lock, int index) throws InterruptedException{
		try {
			lock.lock();
			Thread.sleep(1000);
			 value = index;
			 System.out.println(Thread.currentThread().getName() + ": write value" + value);
		}finally {
			lock.unlock();
		}
	}
	
	public static void main(String[] args) throws Exception {
		final Shit shit = new Shit();
		Runnable readRun = new Runnable() {
			@Override
			public void run() {
				try {
					//shit.handleRead(rLock);
					shit.handleRead(lock);
				}catch(InterruptedException e) {
					e.printStackTrace();
				}
			}
		};
		
		Runnable writeRun = new Runnable() {
			@Override
			public void run() {
				try {
					//shit.handleWrite(rLock, new Random().nextInt());
					shit.handleWrite(lock,  new Random().nextInt());
				}catch(InterruptedException e) {
					e.printStackTrace();
				}
			}
		};
		
		for(int i =0 ; i<18;i++) {
			new Thread(readRun, "T"+i).start();
		}
		
		for(int i = 18; i<20;i++) {
			new Thread(writeRun, "T" + i).start();
		}
	}
}
```
当锁类型为读写锁时，因为读取过程不阻塞，所有线程将同时开始读取数据，同时，写线程在一秒后也开始写，两个写过程互斥，总共用时2秒。
当锁类型为重入锁时，所有线程任意过程都互相阻塞，因此用时大于20秒。










