---
title: 包模块规范：AMD,Commonjs与ES2015
date: 2016-5-19
categories:
- 前端
- js
- es6
tags:
- 前端
- js
- amd
- commonjs
---
# AMD
翻译自[amdjs官方wiki](https://github.com/amdjs/amdjs-api/wiki/AMD)

异步模块定义（AMD）API指定了定义模块的机制，以便模块和它的依赖项可以异步加载。这特别适合于浏览器环境，但其中模块的同步加载会导致性能、可用性、调试和跨域访问问题。实现了AMD的，是requirejs。

### API定义
define(id?, dependencies?, factory);

**id**: 第一个参数id是字符串文字。它指定正在定义的模块的ID。这个参数是可选的，如果它不存在，模块ID应该默认为加载程序请求给定响应脚本的模块的ID。当出现时，模块ID必须是“顶级”或绝对ID（不允许相关ID）。

**dependencies**: 第二个参数依赖项是模块ID的数组文字，它是被定义的模块所需的依赖项。依赖关系必须在模块工厂函数执行之前解决，并且解析的值应该作为参数传递给工厂函数，参数位置对应于依赖数组中的索引。

**factory**: 第三个参数，工厂，是实例化模块或对象时应该执行的函数。如果工厂是一个函数，它应该只执行一次。如果工厂参数是一个对象，则应该将该对象分配为模块的导出值。

例子：
``` javascript
 define("alpha", ["require", "exports", "beta"], function (require, exports, beta) {
       exports.verb = function() {
           return beta.verb();
           //Or:
           return require("beta").verb();
       }
   });
```

# CommonJS
以下翻译自[webpack官方wiki](https://github.com/webpack/docs/wiki/commonjs)。

CommonJS组定义一个模块的格式来解决问题以确保JavaScript范围每个模块是在它自己的命名空间执行。

commonjs提供了两个工具来做这件事情：
1. require()方法, 可以使你向当前scope导入模块。
2. module对象，可以使你从当前scope导出一些东西。

实现了CommonJS的是nodejs,webpack,browserify等。

例子：
``` javascript
// moduleA.js
module.exports = function( value ){
	return value*2;
}

// moduleB.js
var multiplyBy2 = require('./moduleA');
var result = multiplyBy2( 4 );
```

# ES2015

ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代CommonJS和AMD规范，成为浏览器和服务器通用的模块解决方案。对ES2015来说，一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用export关键字输出该变量。下面是一个JS文件，里面使用export命令输出变量。

模块功能主要由两个命令构成：export和import

例子：
``` javascript
// circle.js

export function area(radius) {
  return Math.PI * radius * radius;
}
export function circumference(radius) {
  return 2 * Math.PI * radius;
}

import * as circle from './circle';
```