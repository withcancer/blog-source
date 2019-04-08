---
title: JAVA并发编程之线程池
date: 2016-8-9
categories:
- 后端
- JAVA
tags:
- 后端
- JAVA
- 并发
---
## Executor框架提供了各种类型的线程池，主要有以下几个：
- newFixedThreadPool()：该方法返回一个固定线程数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。
- newSingleThreadExecutor()：该方法返回一个只有一个线程的线程池。当有一个新的任务提交时，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。
<!-- more -->
- newCachedThreadPool()：该方法返回一个根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但如果有空闲线程可以复用，则会优先使用可以复用的线程。如果所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。
- newSingleThreadScheduledExecutor()：该方法返回一个``scheduledExecutorService``对象，线程池大小为1。``ScheduledExecutorService``接口在``ExecutorService``接口之上扩展了在给定时间内执行某人物的功能，如在某个固定的延时之后执行(schedule)，或者周期性(scheduleAtFixedRate, scheduleAtFixedDelay)的执行某个任务。
- newScheduledThreadPool()：该方法也返回一个``ScheduledExecutorService``对象，但是可以指定线程的数量。

- ThreadPoolExcutor()：

  **corePollSize**: 池中所保存的线程数，包括空闲线程，也就是核心池的大小

  **maximumPoolSize**：池中允许的最大线程数
 
  **keepAliveTime**：当线程数量大于``corePoolSize``值时，在没有超过指定的时间内是不从线程池中将空闲线程删除的，如果超过此时间，则删除。
  
  **unit**: ``keepAliveTime``参数的时间单位
  
  **WorkQueue**：执行前用于保持任务的队列。此队列仅保持由``execute``方法提交的``Runnable``任务。
  
  为了更好地理解以上使用的关系，可以进行以下详细化的注释：
  
  A 代表欲执行的``runnable``的数量
    
  B 代表``corePoolSize``
  
  C 代表``maximumPoolSize``
  
  D 代表A-B
  
  E 代表 ``new LinkedBlockingDeque<Runnable>``；队列，无构造参数
  
  F 代表``SynchronousQueue``队列
  
  G 代表``KeepAliveTime``
  
  有几个操作结论：
  
  - 如果A<=B，那么马上创建线程运行这个任务，并不放入扩展队列中，其他参数功能忽略。
  - 如果A>B&&A<=C&&E，则C和G参数忽略，并把D放入E中等待执行。
  - 如果A>B&&&A<=C&&F，则C和G参数有效，马上创建线程运行这些任务，不把D放在F中，D执行完任务后在指定时间后发生超时则将D进行清除。
  - 如果A>B&&A>C&&E，则C和G参数忽略，并把D放入E中等待执行。
  - 如果A>B&&A>C&&E，则处理C的任务，其他任务则不再处理抛出异常。
## Executor接口地invokeAny和invokeAll方法
使用invokeAny()方法，当任意一个线程找到结果之后，立刻终结中断所有线程。
执行结果可能会有以下三种情况：

- 任务都执行成功，使用过第一个任务返回的结果。
- 任务都失败了，抛出Exception，invokeAny方法将抛出ExecutionException。
- 部分任务失败了，会使用第一个成功的任务返回的结果。

使用invokeAll()方法，要么全成功，要么部分失败，中断退出。
运行结果可能出现的情况：

- 全部运行成功
- 部分运行失败，剩余任务被取消
  