---
title: ES6中Object常用API-Object.assign，Object.create， Object.defineProperty
date: 2016-5-10
categories:
- 前端
- js
- ES6
tags:
- 前端
- javascript
- ES6
---
## Object.assign(target, ...sources)
如果目标对象中的属性具有相同的键，则属性将被源中的属性覆盖。后来的源的属性将类似地覆盖早先的属性。`String`类型和`Symbol`类型的属性都会被拷贝。`Object.assign` 会跳过那些值为`null`或`undefined`的源对象。
- 用法：
```
var o1 = { a: 1 };
var o2 = { b: 2 };
var o3 = { c: 3 };

var obj = Object.assign(o1, o2, o3);
console.log(obj); // { a: 1, b: 2, c: 3 }
console.log(o1);  // { a: 1, b: 2, c: 3 }, 注意目标对象自身也会改变。
```
- 避免浅拷贝

`Object.assign()`拷贝的是属性值。假如源对象的属性值是一个指向对象的引用，它也只拷贝那个引用值。
```
let obj1 = { a: 0 , b: { c: 0}};
let obj2 = Object.assign({}, obj1);

obj1.a = 1;
obj1.b.c = 1;
console.log(obj2) // {a:0, b: {c:1}}
```
如果要进行深度克隆，就要先创建一个源对象的副本，再进行assign。
最简单的写法可能是这样：
```
function clone(origin) {
  let originProto = Object.getPrototypeOf(origin);
  return Object.assign(Object.create(originProto), origin);
}
```

- ployfill

根据emca-262规范内对assign过程的描述，可以对这个方法进行ployfill。
1. Let to be ? ToObject(target).
2. If only one argument was passed, return to.
3. Let sources be the List of argument values starting with the second argument.
4. For each element nextSource of sources, in ascending index order, do
   1. If nextSource is undefined or null, let keys be a new empty List.
   2. Else,
      1. Let from be ! ToObject(nextSource).
      2. Let keys be ? from.[[OwnPropertyKeys]]().
5. For each element nextKey of keys in List order, do
   1. Let desc be ? from.[[GetOwnProperty]](nextKey).
   2. If desc is not undefined and desc.[[Enumerable]] is true, then
      1. Let propValue be ? Get(from, nextKey).
      2. Perform ? Set(to, nextKey, propValue, true).
6. Return to.

```
// 省略前面的对source的判断
// function写法
Object.assign || function (target) {
    for (var i = 1; i < arguments.length; i++) {
        var source = arguments[i];
        for (var key in source) {
          if (Object.prototype.hasOwnProperty.call(source, key)) {
            target[key] = source[key];
          }
        }
      }
    return target;
};

// defineProperty写法
Object.defineProperty(Object, "assign", {
    value: function assign(target, varArgs) {
      if (target == null) { // TypeError if undefined or null
        throw new TypeError('Cannot convert undefined or null to object');
      }

      var to = Object(target);

      for (var index = 1; index < arguments.length; index++) {
        var nextSource = arguments[index];

        if (nextSource != null) {
          for (var nextKey in nextSource) {
            if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
              to[nextKey] = nextSource[nextKey];
            }
          }
        }
      }
      return to;
    },
    writable: true,
    configurable: true
  });
```

## Object.create(proto, propertiesObject)
- 用法
使用`Object.create`可以很方便的解决继承问题，也可以用`extends`语法糖解决。
```
let Parent = {
    sayWord: function() {
        return this.word;
    }
}
let child = Object.create(Parent, {
    word: { 
        value: "foo",
        // writable: true,
        // configurable: true,
        // enumerable: true
        // set
        // get
    }
});
console.log(child); // {word: foo}/__proto__.sayWord()
console.log(child.sayWord()); // foo
```
- ployfill
1. If internalSlotsList is not present, set internalSlotsList to a new empty List.
2. Let obj be a newly created object with an internal slot for each name in internalSlotsList.
3. Set obj's essential internal methods to the default ordinary object definitions specified in 9.1.
4. Set obj.[[Prototype]] to proto.
5. Set obj.[[Extensible]] to true.
6. Return obj.
```
// 来自MDN
if (typeof Object.create !== "function") {
    Object.create = function (proto, propertiesObject) {
        if (typeof proto !== 'object' && typeof proto !== 'function') {
            throw new TypeError('Object prototype may only be an Object: ' + proto);
        } else if (proto === null) {
            throw new Error("This browser's implementation of Object.create is a shim and doesn't support 'null' as the first argument.");
        }

        if (typeof propertiesObject != 'undefined') throw new Error("This browser's implementation of Object.create is a shim and doesn't support a second argument.");
        // 实际上还是继承链写法
        function F() {}
        F.prototype = proto;

        return new F();
    };
}
```

## Object.defineProperty((bj, prop, descriptor)
- 用法
```
Object.defineProperty(obj, "key", {
  enumerable: false,
  configurable: false,
  writable: false,
  value: "static",
});
// 写set,get访问器时，不能同时写writable,value属性
Object.defineProperty(obj, "key", {
  enumerable: false,
  configurable: false,
  set: function(newValue){
     console.log(newValue) // foo
     value = 'bar'; // value存在于全局变量
  },
  get: function(){
     return value;
  }
});
console.log(obj); // {}/key,get,set
obj.key = 'foo'
console.log(obj.key) // bar
```
- 几个属性的意义
1. `writable` : 当writable属性设置为false时，不能对这个属性进行赋值运算。严格模式下会报错，普通模式下设置没有作用。
2. `enumerable`定义了对象的属性是否可以在`for...in`循环和`Object.keys()` 中被枚举。
3. `configurable`特性表示对象的属性是否可以被删除，以及除`writable`特性外的其他特性是否可以被修改。