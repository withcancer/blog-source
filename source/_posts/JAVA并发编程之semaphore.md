---
title: JAVA并发编程之semaphore
date: 2016-8-2
categories:
- 后端
- JAVA
tags:
- 后端
- JAVA
- 并发
---
信号量是对锁的扩展，无论是synchronized还是ReetrantLock，一次都只允许一个线程访问一个资源，而信号量却可以指定多个线程，同时访问一个资源。

<!-- more -->
``` java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class Shit implements Runnable {
	final Semaphore semp = new Semaphore(5);

	public static void main(String[] args) throws Exception {
		ExecutorService exec = Executors.newFixedThreadPool(20);
		final Shit shit = new Shit();
		for (int i = 0; i < 20; i++) {
			exec.submit(shit);
		}
	}

	@Override
	public void run() {
		try {
			semp.acquire();
			Thread.sleep(2000);
			System.out.println(Thread.currentThread().getId() + ":done");
			semp.release();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

```
上面这个例子中，将会输出以5个线程为一组的提示文本。