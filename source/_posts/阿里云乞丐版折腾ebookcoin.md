---
title: 阿里云乞丐版折腾ebookcoin
date: 2017-2-10
categories:
- 后端
- 区块链
tags:
- 后端
- 区块链
- nodejs
- ebookcoin
---
阿里云除了ECS外，今年还推出了轻量应用服务器。主要用来部署个人应用，小网站后台，我也申请了一个来玩，三个月135元。

大多数云主机，比如aws，都是没有图形画面的，在网页控制台只能使用简陋的交互式终端来操作。
<!-- more -->
{% asset_img 2.png 简陋的终端 %}

我的目的是在GUI下，用VSCode调试区块链程序-[Ebookcoin（亿书）](https://github.com/Ebookcoin/ebookcoin)。
从一无所有的云服务器中建立调试环境，主要有以下几个步骤：
1. 建立桌面（vncviewer)到云服务器间的连接（vncserver）
2. 在云服务器上安装，配置GUI环境
3. 安装git，VSCode等

## 建立桌面（vncviewer)到云服务器间的连接（vncserver）
首先，要在阿里云防火墙中打开5901和5902端口，否则后面桌面端无法访问。

在远程连接中安装vncserver，执行：
```
sudo apt-get update
sudo apt-get install vnc4server
```
然后执行，启动服务，输入访问密码
```
vncserver
```
在vncviewer中输入公网IP:1即可访问。
## 在云服务器上安装，配置GUI环境
阿里云官方推荐的gnome，ubuntu-desktop图形环境很大，对乞丐版20GB的空间太大，所以换成不足300MB的xfce4。
在远程连接中安装xfce4，执行：
```
sudo apt-get install xfce4
```
因为要在vncviewer中运行GUI，所以不能简单通过`startx`来运行，需要配置`xstartup`，末尾添加
```
sesion-manager & xfdesktop & xfce4-panel &
xfce4-menu-plugin &
xfsettingsd &
xfconfd &
xfwm4 &
```
网上很多文章都说：注释掉`x-window-manager`，但是如果注释掉这一句会导致阿里云浏览器端控制台无法打开，保留并不会影响任何功能。

重新启动vncserver即可。
```
vncserver -kill :1
vncserver
// 带图形参数启动
vncserver -geometry 1280x1024 -depth 16:1
```
vncviewer重新连接，xfce4就出来了。

{% asset_img 1.png xfce4桌面 %}

xfce4默认情况下，两个问题的解决方法：
1. `Tab`失效：修改快捷键设置-Switch window for same application
2. `sudo`时消除`unable to resolve host`，在host内增加阿里云主机名，也就是那一长串英文
## 安装git，安装vscode
```
sudo apt-get install git
wget https://vscode.cdn.azure.cn/stable/0759f77bb8d86658bc935a10a64f6182c5a1eeba/code_1.19.1-1513676564_amd64.deb
sudo dpkg -i code_1.19.1-1513676564_amd64.deb
```
VSCode无法启动时，根据github讨论的结果，可以使用如下方法：
```
sudo sed -i 's/BIG-REQUESTS/_IG-REQUESTS/' /usr/share/code/libxcb.so.1
sudo sed -i 's/BIG-REQUESTS/_IG-REQUESTS/' /usr/share/code/libxcb.so.1.1.0
```
{% asset_img 3.png vscode无法启动的解决讨论 %}
## 调试ebookcoin
现在，基本工具已经具备，开始安装调试工具。
1. 安装nodejs到桌面文件夹apps中
```
wget https://nodejs.org/dist/v8.9.3/node-v8.9.3-linux-x64.tar.xz
xz -d node-v8.9.3-linux-x64.tar.xz
mkdir apps
tar -xvf node-v8.9.3-linux-x64.tar
mv node-v8.9.3-linux-x64 node
mv node apps
// 添加path
export NODE_HOME=/home/admin/Desktop/node
export PATH=$PATH:$NODE_HOME/bin 
export NODE_PATH=$NODE_HOME/lib/node_modules
// /root/.bashrc内添加生效
source /etc/profile
```
2. Clone代码
```
git clone https://github.com/Ebookcoin/ebookcoin.git
// 使用SSH连接github后，clone submodule
git submodule init
git submodule update
```
3. 构建代码
```
// 安装依赖包
cd ebookcoin
npm install
// 全局安装 grunt-cli:
npm install grunt-cli -g
// 全局安装 bower:
npm install bower -g
// 构建前台
cd public
npm install
bower install
grunt release
```
4. 运行
直接在app.js中进行vscode debug，区块链程序已经跑起来了。
