---
title: CSS居中
date: 2016-2-5
categories:
- 前端
- CSS
tags:
- 前端
- CSS
---

# 水平居中
## 块级元素（宽度固定）
有宽度的前提下，可以设置这个块级元素的左右margin为auto来实现水平居中。

<!-- more -->
``` html
<!DOCTYPE HTML>
<html>
<body>
<style>
.container{

}
</style>
<div class="container">
<div style="height:200px;width:200px;margin-left:auto;margin-right:auto;background-color:red"></div>
</div>
</body>
</html>
```
另外，可以利用绝对定位来实现水平居中。首先``left:50%``，然后再``margin``到元素宽度一半的地方。
``` html
<!DOCTYPE HTML>
<html>
<body>
<style>
</style>
<div class="container">
<div style="position:absolute;left:50%;margin-left:-100px;background-color:red"></div>
</div>
</body>
</html>
```
{% asset_img 1.png 块级元素水平居中 %}
## 块级元素（宽度不定）
没有指定宽度时，可以通过``translateX``属性来实现。
``` html
<!DOCTYPE HTML>
<html>
<body>
<style>
.inner{
position:absolute;
left:50%;
transform: translateX(-50%);
}
</style>
<div class="container">
<div class="inner">
这种方法是最不推荐的方法，因为transform属性在各个浏览器中的表现行为不一致，所以会出现一些兼容性的问题，只有当已知用户浏览器时才推荐使用。
</div>
</div>
</body>
</html>
```
{% asset_img 2.png 块级元素水平居中（宽度未定） %}
## 行内元素
块级元素可以通过设置``display:inline-block``来转换成行内元素来达到相同的效果。
设置父容器的``text-align``为``center``即可实现行内元素居中。
``` html
<!DOCTYPE HTML>
<html>
<body>
<style>
.container{
  text-align:center;
}
</style>
<div class="container">

<span style="background-color:red">水平居中或者说水平垂直居中的方案很多种,但在实际当中,不一定.....</span>
</div>
</body>
</html>
```
{% asset_img 3.png 行内元素水平居中（宽度未定）%}
## flex布局
设置父容器的flex布局即可实现各类元素水平居中。
``` html
<!DOCTYPE HTML>
<html>
<body>
<style>
.container{
  display:flex;
  justify-content: center;
}
</style>
<div class="container">
<div style="height:200px;width:200px;background-color:red"></div>
</div>
</body>
</html>
```

# 垂直居中
## 行内元素
inline 元素的行高与``inline-height``相等，则中间内容居中。
``` html
<!DOCTYPE HTML>
<html>
<body>
<style>
.container{
height:300px;
}
.inner{
line-height:300px;
}
</style>
<div class="container">
<span class="inner">
这种方法是最不推荐的方法，因为transform属性在各个浏览器中的表现行为不一致
</span>
</div>
</body>
</html>
```
可以利用inline元素的 CSS 属性 ``vertical-align``，将其设置为 middle，父容器设置为``display:table``。
``` html
<!DOCTYPE HTML>
<html>
<body>
<style>
.container{
display:table;
height:300px;
}
.inner{
display:table-cell;
 vertical-align:middle;

}
</style>
<div class="container">
<span class="inner">
这种方法是最不推荐的方法，因为transform属性在各个浏览器中的表现行为不一致
</span>
</div>
</body>
</html>
```
{% asset_img 4.png 行内元素垂直居中 %}
## 块级元素(知道高度)
有高度的前提下，可以设置这个块级元素的绝对定位实现垂直居中。
``` html
<!DOCTYPE HTML>
<html>
<body>
<style>
.container{

}
.inner{
position:absolute;
height:100px;
top:50%;
margin-top: -50px;
}
</style>
<div class="container">
<div class="inner">
这种方法是最不推荐的方法，因为transform属性在各个浏览器中的表现行为不一致
</div>
</div>
</body>
</html>
```
## 块级元素（高度未知）
不知道高度时，与水平对齐同理，可以使用``translateY``来实现垂直居中。
``` html
<!DOCTYPE HTML>
<html>
<body>
<style>
.container{

}
.inner{
position:absolute;
top:50%;
tranform:translateY(-50%);
}
</style>
<div class="container">
<div class="inner">
这种方法是最不推荐的方法，因为transform属性在各个浏览器中的表现行为不一致，所以会出现一些兼容性的问题，只有当已知用户浏览器时才推荐使用。
</div>
</div>
</body>
</html>
```
{% asset_img 5.png 块级元素垂直居中（高度未知） %}
## flex布局
设置父容器的flex布局即可实现各类元素垂直居中。
``` html
<!DOCTYPE HTML>
<html>
<body>
<style>
.container{
  height:300px;
  display:flex;
  align-items: center;
}
</style>
<div class="container">
<div>这种方法是最不推荐的方法，因为transform属性在各个浏览器中的表现行为不一致</div>
</div>
</body>
</html>
```
