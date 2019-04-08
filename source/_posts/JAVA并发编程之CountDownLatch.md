---
title: JAVA并发编程之CountDownLatch
date: 2016-8-3
categories:
- 后端
- JAVA
tags:
- 后端
- JAVA
- 并发
---

对于倒计时器锁，一种典型的应用就是火箭发射。火箭发射前，要等待所有的检测线程完工。CountDownLatch接受一个整数作为参数，即当前计数器的计数个数。

下面的这个例子演示了一个模拟的火箭发射过程：
<!-- more -->
``` java
package fuck;

import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Shit implements Runnable{
	
	private static CountDownLatch cdLatch = new CountDownLatch(10);
	private int value;
	
	public static void main(String[] args) throws Exception {
		final Shit shit = new Shit();
		ExecutorService exec = Executors.newFixedThreadPool(10);
		for(int i=0;i<10;i++) {
			exec.submit(shit);
		}
		cdLatch.await();
		System.out.println("Fire!");
		exec.shutdown();
	}
	
	@Override
	public void run() {
		try {
			Thread.sleep(new Random().nextInt(10)*1000);
			System.out.println("Check Over !");
			cdLatch.countDown();
		}catch(InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```
调用``cdLatch.await()``后，该线程将会等待子线程``countdown``完毕后才能执行。