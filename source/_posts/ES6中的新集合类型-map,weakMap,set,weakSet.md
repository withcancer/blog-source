---
title: ES6中的新集合类型-map, weakMap,set,weakSet
date: 2016-5-16
categories:
- 前端
- ES6
tags:
- 前端
- javascript
- ES6
---
# Map
*ecma-262* 规范中，23节 *keyed Collection* 对`Map`有如下定义：
> Map objects are collections of key/value pairs where both the keys and values may be arbitrary ECMAScript language values. A distinct key value may only occur in one key/value pair within the Map's collection. Distinct key values are discriminated using the SameValueZero comparison algorithm.

`Map`对象是key-value的集合，key和value可以是es的任意类型。key值具有唯一性，分辨一个key是不是唯一的，使用的是严格比较法。
<!-- more -->

> The Map constructor is designed to be subclassable. It may be used as the value in an extends clause of a class definition. Subclass constructors that intend to inherit the specified  Map behaviour must include a super call to the Map constructor to create and initialize the subclass instance with the internal state necessary to support the Map.prototype built-in methods.

`Map`构造函数被设计成可以继承的，子类可以用`extends`语句来进行扩展。子类的构造函数在对`Map`进行继承的时候，必须要包含一个对`Map`构造函数的super call，这样就可以使用`Map.prototype`的内置方法。

### API Methods：
``` javascript
Map.prototype.clear
Map.prototype.delete
Map.prototype.entries
Map.prototype.forEach(callbackfn[,thisArg])
Map.prototype.get(key)
Map.prototype.has(key)
Map.prototype.keys
Map.prototype.set(key,value)
Map.prototype.size
Map.prototype.values()
```
- key相同的判断：

在内存地址不一样时，值无法被取到
```
const map = new Map();
map.set(['a'], 1);
map.set(['a'], 2);
console.log(map.get(['a'])); // 此处的['a']的内存地址和之前的都不一样，虽然值相等，仍然get不到值
```
严格比较法带来的现象
```
const map = new Map();
map.set(-0, 1);
map.get(+0); // 是可以get到的
map.set(undfined, 1);
map.get(null); // get不到
```
# WeakMap
*ecma-262* 规范中，23节 *keyed Collection* 对`WeakMap`有如下定义：

> WeakMap objects are collections of key/value pairs where the keys are objects and values may be arbitrary ECMAScript language values. A WeakMap may be queried to see if it contains a key/value pair with a specific key, but no mechanism is provided for enumerating the objects it holds as keys. If an object that is being used as the key of a WeakMap key/value pair is only reachable by following a chain of references that start within that WeakMap, then that key/value pair is inaccessible and is automatically removed from the WeakMap. WeakMap implementations must detect and remove such key/value pairs and any associated resources.

`WeakMap`对象是key-value的集合，key必须是object（不能是基本数据类型）而value可以是es支持的任意类型。`WeakMap`可以查询是不是含有特定key，但是没有对key进行枚举的机制。如果一个作为`WeakMap`的key的object只能被WeakMap自己所引用，那这个key-value pair就会被自动移除。`WeakMap`的实现必须自主检测这类key-value pair有没有引用者，并及时清除。

### API Methods：
```
WeakMap.prototype.delete
WeakMap.prototype.get
WeakMap.prototype.has
WeakMap.prototype.set
```

可以看出`WeakMap`和`Map`有以下的不同：
1. 由于没有枚举，弱引用的特性，`WeakMap`只有四个方法`delete,get,has,set`
2. `WeakMap`每个键对自己所引用对象的引用是 "弱引用",GC在计算对象引用数量的时候并不会把弱引用计算进去。这样当一个对象除了WeakMap没有其他引用的时候就会被GC回收掉。 只要没有外部引用，
3. `WeakMap`只能接受`object`作为键

WeakMap可以用来避免内存泄漏，存储那些将来可能会消失的对象作为键。

# Set & WeakSet
*ecma-262* 规范中，23节 *keyed Collection* 对`Set`有如下定义：
```
Set objects are collections of ECMAScript language values. A distinct value may only occur once as an element of a Set's collection. Distinct values are discriminated using the SameValueZero comparison algorithm.

Set objects must be implemented using either hash tables or other mechanisms that, on average, provide access times that are sublinear on the number of elements in the collection. The data structures used in this Set objects specification is only intended to describe the required observable semantics of Set objects. It is not intended to be a viable implementation model.
```
`Set`对象是es任意类型的集合。每个值都都只能出现一次。分辨唯一值用的是严格比较法。

`Set`实现时采用hash表或是其它机制，提供一种随着集合规模扩大而线性发展的访问原则。`Set`对象内的数据结构只是用来反映出必要的可观察的语义，而并不是用来作为一种可行的实现模型。

### API Methods:
```
Set.prototype.add
Set.prototype.clear
Set.prototype.delete
Set.prototype.entries
Set.prototype.forEach
Set.prototype.has(value)
Set.prototype.keys Set.prototype.size Set.prototype.values
```
特性如同传统编程意义上的`Set`,和`Map`类似有key相同判断特性。
同样的，`WeakSet`和`WeakMap`也有相同的特性，即:
1. 只能add对象而不能是其它原始数据类型
2. `WeakSet`对对象的引用是弱引用，所以`WeakSet`不能枚举，也没有`size`。
3. 除此之外，以上所说的所有类型，都可以接受可迭代对象作为初始化参数，因为它们都内建了Sympol.iterator方法。