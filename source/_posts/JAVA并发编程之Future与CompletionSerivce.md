---
title: JAVA并发编程之Future与CompletionSerivce
date: 2016-8-9
categories:
- 后端
- JAVA
tags:
- 后端
- JAVA
- 并发
---
``Future``调用``get()``方法时，是阻塞的，也就是如果调用``Future``对象的``get()``方法时，任务尚未完成，则调用``get()``方法时会一直阻塞到此任务完成为止。如果是这样的结果，则前面先执行的任务一旦耗时很多，则后面调用``get()``方法就成为了阻塞状态，排队进行等待，大大影响运行效率。主线程不能保证首先获得的是最先完成任务的返回值，这就是``Future``的缺点。
<!-- more -->
下面用一个例子来演示``Future``批处理的缺点。

``` java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class Shit implements Callable<String> {
	private String username;
	private long sleepValue;

	Shit(String username, long sleepValue) {
		this.username = username;
		this.sleepValue = sleepValue;
	}

	@Override
	public String call() throws Exception {
		System.out.println(username);
		Thread.sleep(sleepValue);
		return "Result : " + username;
	}

	@SuppressWarnings({ "unchecked", "rawtypes" })
	public static void main(String args[]) {
		List<Callable> callableList = new ArrayList<Callable>(
				Arrays.asList(new Shit("username1", 5000), new Shit("username2", 4000), new Shit("username3", 3000),
						new Shit("username4", 2000), new Shit("username5", 1000)));
		List<Future> futureList = new ArrayList<Future>();
		ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 10, 5, TimeUnit.SECONDS, new LinkedBlockingDeque());
		for (int i = 0; i < 5; i++) {
			futureList.add(executor.submit(callableList.get(i)));
		}
		try {
			for (int i = 0; i < 5; i++) {
				System.out.println(futureList.get(i).get() + " : " + System.currentTimeMillis());
			}
		} catch (InterruptedException | ExecutionException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
/* 输出：
username1
username5
username4
username3
username2
由于第一个任务耗时较长，因此后面任务的get()被阻塞。
Result : username1 : 1520214428110
Result : username2 : 1520214428110
Result : username3 : 1520214428110
Result : username4 : 1520214428110
Result : username5 : 1520214428110
*/
```
``CompletionSerivce``可以解决这个批处理等待问题。也就是哪个任务先执行完，``CompletionSerivce``就先取得这个任务的返回值再处理。

``` java
public static void main(String args[]) {
		List<Callable> callableList = new ArrayList<Callable>(
				Arrays.asList(new Shit("username1", 5000), new Shit("username2", 4000), new Shit("username3", 3000),
						new Shit("username4", 2000), new Shit("username5", 1000)));
		
		ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 10, 5, TimeUnit.SECONDS, new LinkedBlockingDeque());
		
		CompletionService cs = new ExecutorCompletionService(executor);
		
		for (int i = 0; i < 5; i++) {
			cs.submit(callableList.get(i));
		}
		try {
			for (int i = 0; i < 5; i++) {
			    // take可以阻塞式地获得下一个任务的future
			    // poll方式则是非阻塞式地获得下一个任务的future
				System.out.println(cs.take().get() + " : " + System.currentTimeMillis());
			}
		} catch (InterruptedException | ExecutionException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
/* 输出
username2
username5
username4
username3
username1
Result : username5 : 1520215083204
Result : username4 : 1520215084203
Result : username3 : 1520215085203
Result : username2 : 1520215086203
Result : username1 : 1520215087203
*/
```
