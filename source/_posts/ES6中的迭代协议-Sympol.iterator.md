---
title: ES6的迭代协议-Sympol.iterator
date: 2016-5-20
categories:
- 前端
- js
- ES6
tags:
- 前端
- javascript
- ES6
---
- `Array`和`Map`及`Set`,`String`有内建的可迭代协议，从chrome上可以看出这一点，所以可以使用各类迭代方法对其中的成员进行访问。
- 如果要让没有迭代功能的对象具有迭代性，需要实现`Sympol.iterator`方法。

有以下方式来实现可迭代。
1. 传统方式,缺点是无法使用`for-of`，只能用`next()`进行迭代
```
function Iterator(array){
    var nextIndex = 0;
    
    return {
       next: function(){
           return nextIndex < array.length ?
               {value: array[nextIndex++], done: false} :
               {done: true};
       }
    };
}

let it = Iterator(['foo', 'bar']);

console.log(it.next().value); // 'foo'
console.log(it.next().value); // 'bar'
console.log(it.next().done);  // true
```
2. Sympol.iterator方式，在这种方式下，没有办法使用类似next()方法来进行迭代
```
const Iterator = {
    [Symbol.iterator]() {
        let step = 0;
        const iterator = {
            next() {
                if (step <= 2) {
                    step++;
                }
                switch (step) {
                    case 1:
                        return { value: 'foo', done: false };
                    case 2:
                        return { value: 'bar', done: false };
                    default:
                        return { value: undefined, done: true };
                }
            }
        };
        return iterator;
    }
};
// 解构为 foo bar
console.log(...Iterator);
// 迭代为 foo bar
for(let c of Iterator){
   console.log(c);
}
// 得到 ['f','o','o','b','a','r']
Array.from(Iterator);     
```
3. Generator形式，与es6配合最为紧密，支持方法最多
```
function* Iterator(array){
    var nextIndex = 0;
    
    while(nextIndex < array.length){
        yield array[nextIndex++];
    }
}

var gen = Iterator(['foo', 'bar']);
// 得到IteratorResult（chrome,firefxo) {value: 'foo', done: false}
console.log(gen.next()); 
console.log(gen.next().value); // bar
// 得到 'foo', 'bar'
console.log(...Iterator(['foo', 'bar']));
// 得到 'foo', 'bar'
for(let c of gen){
    console.log(c);
}
// 得到 ['foo', 'bar']
Array.from(gen);
```
另外还有一种无限迭代方式
```
function* idMaker(){
    var index = 0;
    while(true)
        yield index++;
}

var gen = idMaker();

console.log(gen.next().value); // '0'
console.log(gen.next().value); // '1'
console.log(gen.next().value); // '2'
```