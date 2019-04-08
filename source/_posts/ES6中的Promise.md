---
title: ES6中的Promise
date: 2016-4-28
categories:
- 前端
- ES6
tags:
- 前端
- javascript
- ES6
- Promise
---
Promise 对象是一个代理对象（代理一个值），被代理的值在Promise对象创建时可能是未知的。它允许你为异步操作的成功和失败分别绑定相应的处理方法（handlers）。 这让异步方法可以像同步方法那样返回值，但并不是立即返回最终执行结果，而是一个能代表未来出现的结果的promise对象

一个 Promise有以下几种状态:

- pending: 初始状态，既不是成功，也不是失败状态。
- fulfilled: 意味着操作成功完成。
- rejected: 意味着操作失败。
<!-- more -->
## 方法
``` javascript
Promise.all(iterable)
Promise.race(iterable)
Promise.reject(reason)
Promise.resolve(value)
Promise.prototype.catch()
Promise.prototype.then()
```
### Promise.all 

等待所有代码的完成（或第一个代码的失败）。它的参数是一组iterable。某个值如果不是promise,也会被按顺序计入结果集中。
``` javascript
let getAblumFromSinger = new Promise((resolve, reject) => {
    setTimeout(()=>{
        console.log("getAblumFromSinger");
    }, 1500);
    resolve("AblumIsHere");
});

let getSongFromAblum = new Promise((resolve, reject)=>{
    setTimeout(()=>{
        console.log("getSongFromAblum");
    }, 1000);
    resolve("SongIsHere");
});

let rejectPromise = new Promise((resolve, reject)=>{
    reject("RejectIsHere");
})

Promise.all([getAblumFromSinger,getSongFromAblum]).then(values => { 
    console.log(values); // 执行上异步，getSongFromAblum比getAblumFromSinger先执行，但是结果是按任务顺序排列的，当promise中有reject时，将会立刻失败
});
```
### Promise.prototype.catch
如果 onRejected 抛出一个错误或返回一个失败的 Promise ，Promise  通过catch()返回失败结果；否则，它将显示为成功。 
### Promise.prototype.then
then() 方法返回一个  Promise 。它最多需要有两个参数：Promise 的成功和失败情况的回调函数。

``` javascript
Promise.resolve()
  .then( () => {
    // 使 .then() 返回一个 rejected promise
    throw 'Oh no!';
  })
  .catch( reason => {
    console.error( 'onRejected function called: ', reason );
  })
  .then( () => {
    console.log( "I am always called even if the prior then's promise rejects" );
  });
```
### Promise.race(iterable) 
Promise.race(iterable) 方法返回一个 promise ，并伴随着 promise对象解决的返回值或拒绝的错误原因, 只要 iterable 中有一个 promise 对象"解决(resolve)"或"拒绝(reject)"。
``` javascript
var p1 = new Promise(function(resolve, reject) { 
    setTimeout(resolve, 500, "one"); 
});
var p2 = new Promise(function(resolve, reject) { 
    setTimeout(resolve, 100, "two"); 
});

Promise.race([p1, p2]).then(function(value) {
  console.log(value); // "two"
  // 两个都完成，但 p2 更快，这点与all不同，all会按顺序返回所有的任务结果
});
```
### Promise.resolve
### Promise.reject
返回拒绝或接受promise结果