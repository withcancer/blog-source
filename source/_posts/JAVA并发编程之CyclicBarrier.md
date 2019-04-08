---
title: JAVA并发编程之CyclicBarrier
date: 2016-8-10
categories:
- 后端
- JAVA
tags:
- 后端
- JAVA
- 并发
---

CyclicBarrier可以用于多线程计算数据,最后合并计算结果的场景。
<!-- more -->
``` java
import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Shit implements Runnable {
	private static ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<String, Integer>();
    private static CyclicBarrier barrier;
	public static void main(String[] args) throws Exception {
		    barrier = new CyclicBarrier(5, new Runnable() {
			// 栅栏动作，在计数器为0的时候执行
			@Override
			public void run() {
				System.out.println("所有得分得到！");
				int result = 0;
				for (String key : map.keySet()) {
					result += map.get(key);
				}
				System.out.println("平均值为：" + (result/5));
			}
		});

		ExecutorService es = Executors.newCachedThreadPool();
		for (int i = 0; i < 5; i++) {
			es.execute(new Shit());
		}
	}

	@Override
	public void run() {
		try {
			int score = new Random().nextInt(100);
			map.put("科目" + Thread.currentThread().getName() , score);
			System.out.println("科目" +  Thread.currentThread().getName() + "已获得得分: " + score);
			Thread.sleep(1000);
			barrier.await();
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (BrokenBarrierException e) {
			e.printStackTrace();
		}
	}
}
```
只有当五个线程都结束时，才会进入计算平均值的方法中，CyclicBarrier起到了一个栅栏作用，一旦线程都达到了栅栏处，就会进入闸门方法中。