---
title: ES6中Object密封API-Object.preventExtensions,Object.seal,Object.freeze
date: 2016-5-14
categories:
- 前端
- ES6
tags:
- 前端
- javascript
- ES6
---
## Object.preventExtensions
可以使得一个对象无法添加新属性，但是可以改变原来的属性
- 用法
``` javascript
let obj = {foo: 'bar'};
Object.preventExtensions(obj);
obj.name = 'test';
// undefined，严格模式下报错
console.log(obj.test);
```
<!-- more -->
另外，Object.isExtensible() 方法可以判断一个对象是否是可扩展的。
## Object.seal
密封一个对象是让一个对象不能添加新的属性，不能删除已有属性，以及不能修改已有属性的可枚举性、可配置性、可写性，但可能可以修改已有属性的值。
- 用法
``` javascript
'use strict';
let obj = {};
Object.defineProperty(obj, 'foo', {writable: true});
let o = Object.seal(obj);
o.key = 'fff'; // 增加新属性报错
o.foo = 'fff'; // 修改既有属性的值
delete o.foo; // 删除既有属性报错
Object.defineProperty(obj, 'foo', {writable: false}); // TODO: writable，configurable从true改成false可以，但是不能反过来
```
另外，Object.isSealed() 方法可以判断一个对象是否是被冻结的。
## Object.freeze
- 用法

Object.freeze方法可以使得一个对象无法添加新属性、无法删除旧属性、也无法改变属性的值，使得这个对象实际上变成了常量。
``` javascript
let obj = {};
Object.defineProperty(obj, 'foo', {writable: true});
let o = Object.freeze(obj);
o.key = 'fff'; // 增加新属性报错
o.foo = 'fff'; // 修改既有属性的值
delete o.foo; // 删除既有属性报错
Object.defineProperty(obj, 'foo', {writable: false}); // TODO: writable，configurable从true改成false可以，但是不能反过来
```


