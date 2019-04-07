---
title: 真正理解nodejs中的EventLoop
date: 2016-6-1
categories:
- 前端
- js
- nodejs
tags:
- 前端
- js
- nodejs
- eventloop
---
nodejs是基于的事件的平台。这意味着node中发生的一切都是对事件的反应。事务通过node遍历一个级联的回调。在node.js里，任何异步方法（除timer,close,setImmediate之外）完成时，都会将其callback加到poll queue里,并立即执行。

远离开发层面，这些功能都是由一个叫做libuv的库提供的。

Eventloop可能是nodejs中最被人所误解的一个概念。

## 一般的错误认识
### 错误认识: eventloop运行于一个单独的相对独立于用户代码的线程中

- 错误：eventloop运行于一个相对独立于用户代码的线程中。有一个专门运行eventloop的线程，同时有一个运行用户代码的主线程。每当有异步操作时，主线程将工作交给事件循环线程，一旦完成，事件循环线程将ping主线程执行回调。
- 正确：只有一个线程执行JavaScript代码，这是事件循环运行的线程。回调函数的执行（Node.js中每个用户的代码都是一个回调）是由事件循环做。稍后会更深入地讨论这个问题。

### 错误认识：异步操作由一个线程池处理

- 错误：异步操作，如IO工作，HTTP请求或与数据库总是由libuv线程池来装载。
- 正确：libuv默认情况下创建一个有四个线程的线程池线来加载异步工作。今天的操作系统已经为许多I/O任务提供异步接口。只要有可能，libuv将使用这些异步接口，避免线程池的使用。这同样适用于像数据库这样的第三方子系统。驱动程序的作者宁愿使用异步接口，也不愿意使用线程池。简言之：只有在没有其他方法的情况下，线程池才会被用于异步I/O。

### 错误认识：eventloop就像栈或队列

- 错误：事件循环按FIFO不断地遍历异步任务，并在任务完成时执行回调。
- 虽然有类似队列的结构，但事件循环并没有完全类似于栈。事件循环作为一个过程，是一组由特定任务组成的按循环方式处理的过程。

## 理解事件循环处理过程

要真正理解事件循环，我们必须了解哪些工作是在哪个阶段完成的。下图展示了这一过程：


这一过程由以下几个部分组成：

### 计时器
任何由setTimeout()或setInerval()定时的操作都会在这里执行。
### IO回调
这里大部分的回调函数将被处理。所有的用户代码在Node.js基本上是在回调（例如一个回调函数传入的HTTP请求会触发一个级联的回调），这是用户的代码。
### IO轮询
对下一次运行将要处理的新事件的轮询。
有两个主要方法：
- 执行下限时间已经达到的timers的回调
- 处理poll队列中的事件

当事件循环进入poll阶段，并且没有timers被设定时，将发生两种情况之一：

- 如果poll队列不空，事件循环将遍历其回调执行同步直到队列中的回调都被调用，或者达到系统上限。
- 如果poll队列是空的，那么两个事件中的一个将会发生：
- - 如果脚本被setImmdiate()定时器定时，eventloop将会结束poll阶段，并进入下一个阶段
- - 如果代码不被setimmediate()定时器定时，事件循环会等待回调被添加到队列中，然后立刻执行它们。

如果poll队列为空，eventloop将会检查哪些timers的已经到期。如果有一个或多个timer到期，eventloop将会回到timers阶段去执行回调。

### Set Immediate
运行所有由setImmediate()发起的操作。
### Close
这里是所有的(close)事件回调的处理。

例如有以下代码：
``` javascript
const fs = require('fs');

function someAsyncOperation(callback) {
  // 假设需要95ms
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

someAsyncOperation(() => {
  const startCallback = Date.now();
  
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```
console最终的输出为105毫秒，它的过程是这样的：
1. 当eventloop来到poll阶段时，它的队列为空，因为此时readFile还没有结束。所以它将等待最新的timer到期。
2. 95ms后，readFile结束，callback执行还需要另外的10ms。
3. callback执行结束，因为队列中已经没有了其他的callback,它将回到timers阶段，setTimeOut()执行callback，得到105ms。

