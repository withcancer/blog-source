---
title: 143行js顶部进度条最小插件-nanobar.js源码解析
date: 2017-2-7
categories:
- 前端
- javascript
tags:
- 前端
- javascript
- 源码
- dom
- nanobar
---
网页顶部进度条插件的有四五种，基本原理就是动态地创建一个元素，然后通过设置它的width来实现动画效果，width增长到达指定位置时，将其去掉。
来看看nanobar.js作者[jacoborus](https://github.com/jacoborus)是怎么做到的吧！

<!-- more -->
```javascript
/* http://nanobar.micronube.com/  ||  https://github.com/jacoborus/nanobar/    MIT LICENSE */
(function (root) {
  'use strict'
  // container styles
  var css = '.nanobar{width:100%;height:4px;z-index:9999;top:0}.bar{width:0;height:100%;transition:height .3s;background:#000}'

  // add required css in head div
  function addCss () {
    var s = document.getElementById('nanobarcss')

    // check whether style tag is already inserted
    if (s === null) {
      s = document.createElement('style')
      s.type = 'text/css'
      s.id = 'nanobarcss'
      document.head.insertBefore(s, document.head.firstChild)
      // the world
      if (!s.styleSheet) return s.appendChild(document.createTextNode(css))
      // IE
      s.styleSheet.cssText = css
    }
  }

  function addClass (el, cls) {
    if (el.classList) el.classList.add(cls)
    else el.className += ' ' + cls
  }

  // create a progress bar
  // this will be destroyed after reaching 100% progress
  function createBar (rm) {
    // create progress element
    var el = document.createElement('div'),
        width = 0,
        here = 0,
        on = 0,
        bar = {
          el: el,
          go: go
        }

    addClass(el, 'bar')

    // animation loop
    function move () {
      var dist = width - here

      if (dist < 0.1 && dist > -0.1) {
        place(here)
        on = 0
        if (width === 100) {
          el.style.height = 0
          setTimeout(function () {
            rm(el)
          }, 300)
        }
      } else {
        place(width - dist / 4)
        setTimeout(go, 16)
      }
    }

    // set bar width
    function place (num) {
      width = num
      el.style.width = width + '%'
    }

    function go (num) {
      if (num >= 0) {
        here = num
        if (!on) {
          on = 1
          move()
        }
      } else if (on) {
        move()
      }
    }
    return bar
  }

  function Nanobar (opts) {
    opts = opts || {}
    // set options
    var el = document.createElement('div'),
        applyGo,
        nanobar = {
          el: el,
          go: function (p) {
            // expand bar
            applyGo(p)
            // create new bar when progress reaches 100%
            if (p === 100) {
              init()
            }
          }
        }

    // remove element from nanobar container
    function rm (child) {
      el.removeChild(child)
    }

    // create and insert progress var in nanobar container
    function init () {
      var bar = createBar(rm)
      el.appendChild(bar.el)
      applyGo = bar.go
    }

    addCss()

    addClass(el, 'nanobar')
    if (opts.id) el.id = opts.id
    if (opts.classname) addClass(el, opts.classname)

    // insert container
    if (opts.target) {
      // inside a div
      el.style.position = 'relative'
      opts.target.insertBefore(el, opts.target.firstChild)
    } else {
      // on top of the page
      el.style.position = 'fixed'
      document.getElementsByTagName('body')[0].appendChild(el)
    }

    init()
    return nanobar
  }

  if (typeof exports === 'object') {
    // CommonJS
    module.exports = Nanobar
  } else if (typeof define === 'function' && define.amd) {
    // AMD. Register as an anonymous module.
    define([], function () { return Nanobar })
  } else {
    // Browser globals
    root.Nanobar = Nanobar
  }
}(this))
```
---
## 大体看下来，这个插件有这样几个特点：

- **dom+js原生选择器**
- **支持模块化**
- **es5+IIFE**
- **不用分号派**

---

## 详细来看：

### 在程序的开头，定义了必要的Css属性，包括bar（主体）和Nanobar（容器）两个class：

```javascript
.nanobar{
width:100%;
height:4px;
z-index:9999;
top:0
}

.bar{
width:0;
height:100%;
transition:height .3s;
background:#000}
```
从css内容来看，仅有.bar有``transition:height .3s``的过渡设置，``height``过渡发生的时间应该是被删除时。在横向应该是没有动画效果，但是从官网演示效果来看，横向仍然有一定的动画效果，这个问题下面会提到。

另外，引用作者原话：

> Nanobar injects a style tag in your HTML head. Bar divs has class .bar, and its containers .nanobar, so you can overwrite its values.

> You should know what to do with that ;)

### 然后来看构造函数NanoBar：

NanoBar接受一个opts作为参数，文档记载的opts详细内容如下：

名称|功能
--|--
id|指定nanobar的id
classname|指定nanobar的class
target|指定Nanobar的表示位置，一般对于做顶部进度条来说不到。值得一提的是，这个参数类型为*DOM Element*，你必须使用``document.getxxxxx``之类的方法为其赋值。

#### 首先声明了三个变量：

名称|描述
--|--
el|这就是动态创建的元素-一个既没有ID也没有Class的空div
applyGo|进度条移动的方法
nanobar|nanobar对象，它将在new构造函数时作为结果返回

其中，nanobar包含这两个元素：

名称|描述
--|--
el|上面动态创建的元素
go|对外开放的方法，参数为数值，那么它肯定代表了百分比而不是像素等实际物理单位

此处的go处理内实质上调用的是applyGo，而applyGo此时肯定为``undefined``，所以applyGo实际上在别处赋值。这样处理的结果，相当于是一层封装，隐藏了内部实际的go方法内容。

另外也可以猜测，nanobar的最简单的使用方法如下：
```javascript
var nanobar = new Nanobar();
nanobar.go(80);
```
#### 接下来，声明了两个内部函数，这两个内部函数可以访问上面提到的三个变量：

名称|作用
--|--
rm|用于进度完成后，删除动态创建的元素
init|初始化方法，这个需要重点关注

然后是一些必要处理，由这三个部分组成：

1. ``addCss``方法，为``head``节点内增加``<style id="nanobarcss">``节点，并把上文的css填入其中。
2. 调用``addClass``方法，创建类名为``nanobar``的容器。需要注意的是，相比于直接操作``className``方法内调用了HTML5的新API``classList``，使用它可以像jquery的addClass、removeClass一样方便的对dom对象的class进行增加删除判断。更多信息请看[这里](https://developer.mozilla.org/en-US/docs/Web/API/Element/classList)。
3. 接下来是对``opts``参数进行处理：
主要是为el元素赋予id和className，根据是否指定了父容器，也就是``target``，改变容器的position，并且最终将它插入到对应的位置上。

#### 接着来看init()方法：

前面所有的操作，创建了一个名为``nanobar``的容器，接下来就该创建``bar``主体了。

可以看到，``bar``变量内仍然和``nanobar``一样，由``el``和``go``两部分组成，``go``最终将被赋值到外层容器的``applyGo``，``el``将被作为子元素插入到外层容器的``el``内。

这样，当用最简单的方式调用go时，它的顺序就是这样的：

## 容器nanobar.go->applyGo->本体bar.go

---

#### 那么调用了go方法后，为什么横向会有一定的动画效果呢？

观察一下nanobar的动作方法``go``、``move``、``place``

其中的控制量有这么几个：

名称|作用
--|--
on|相当于布尔flag，标识了进度是否完成了
here|终点位置
dist|与终点相比的距离

实际处理流程可以这样表示：

```flow
place(width - dist / 4) -> dist < 0.1
-> dist = width -here -> 高度置零，删除元素
```
形成动画的根本原因则是这么两个原因：

1. 方法``place(width - dist / 4)``对剩余空间的细分
2. 第58紧随其后的``setTimeout(go,16)``，假设把x轴看成是16ms，把Y轴看成是每次细分的长度，将会得到一个图像类似于log2x(前期趋势大，后期趋势平稳，类似于动画函数中的``ease-out``)的表达式。中学都学过，就不再赘述了。