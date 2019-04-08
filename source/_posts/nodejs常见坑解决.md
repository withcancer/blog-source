---
title: nodejs常见坑解决
date: 2016-5-19
categories:
- 前端
- nodejs
tags:
- 前端
- javascript
- nodejs
---
1. node-sass包内安装失败
``` javascript
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install node-sass

// angular全局安装出错时手动替换node-sass
C:\Users\xxxx\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\node-sass\vendor\win32-x64-57\
下的binding.node，换成从网上下载的win32-x64-57_binding.node
```
<!-- more -->
2. 换源
```
npm config set registry=https://registry.npm.taobao.org
```