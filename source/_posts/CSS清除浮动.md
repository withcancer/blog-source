---
title: CSS清除浮动
date: 2016-2-2
categories:
- 前端
- CSS
tags:
- 前端
- CSS
---
首先定义如下几个div。
``` HTML
<!DOCTYPE HTML>
<html>
<body>
<style>
.container{
  border: 1px solid #c5c5c5;
}
</style>
<div class="container clearfix">
<div id="first" style="width:100px;height:100px;background-color:yellow;float:left;"></div>
<div style="width:100px;height:200px;background-color:red;float:left"></div>
</div>
</body>
</html>
```
<!-- more -->
此时可以发现，两个``div``元素行内对齐排列，加了浮动之后的元素脱离了标准流，所以父容器出现了高度塌陷。
{% asset_img 1.jpg 图1 %}
清除浮动的方法有以下几个：
- 增加一个空的``div``元素
``` html
<div style="clear:both"></div>
```
{% asset_img 2.jpg 图2 %}
- 为父级元素增加``overflow:hidden``属性。
``` css
.container{
  border: 1px solid #c5c5c5;
  overflow: hidden;
}
```
{% asset_img 3.jpg 图3 %}
- 为父级元素增加伪元素。
``` css
.clearfix:before,
.clearfix:after {
    display: table;
    content: " ";
}
.clearfix:after {
    clear: both;
}
.clearfix{
    *zoom: 1;
}
```
{% asset_img 4.jpg 图4 %}