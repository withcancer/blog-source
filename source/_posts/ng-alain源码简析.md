---
title: ng-alain源码简析
date: 2016-5-20
categories:
- 前端
- angular
tags:
- 前端
- 源码
- ng-alain
---
# 外围配置

## Dockerfile
```
# STEP 1: Build
// copy package.json和lock到当前工作目录
// node执行install,把创建好的./node_modules复制到新创建的ng-alain文件夹下
// 设置工作目录为/ng-alain
// 拷贝所有源文件到工作目录中
FROM node:8-alpine as builder

LABEL authors="cipchk <cipchk@qq.com>"

COPY package.json package-lock.json ./

RUN npm set progress=false && npm config set depth 0 && npm cache clean --force
RUN npm i && mkdir /ng-alain && cp -R ./node_modules ./ng-alain

WORKDIR /ng-alain

COPY . .

RUN npm run build

# STEP 2: Setup
// 拷贝nginx配置，ssl配置到nginx中
// 拷贝/ng-alain下的dist目录到nginx文件夹
// 运行nginx对页面进行代理
FROM nginx:1.13.5-alpine

COPY --from=builder /ng-alain/_nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /ng-alain/_nginx/ssl/* /etc/nginx/ssl/

RUN rm -rf /usr/share/nginx/html/*

COPY --from=builder /ng-alain/dist /usr/share/nginx/html

CMD [ "nginx", "-g", "daemon off;"]
```
<!-- more -->
## nginx配置
配置了端口80，虚拟主机名localhost，主目录/usr/share/nginx/html，首页index.html，错误处理画面50x.html
``` config
server {
    listen       80;
    # listen 443;
    # ssl on;
    # ssl_certificate /etc/nginx/ssl/server.crt;
    # ssl_certificate_key /etc/nginx/ssl/server.key;

    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
## docker compose
简单的使用上面的dockerfile进行build，并传入环境变量NODE_ENV:production

# 结构

ng-alain由以下几大模块组成：
- core
- layout
- routes
- shared

## core模块
core模块首先提供了i18n服务，下面来看ng-alain是如何实现i18n转换的：

1. I18NService实现了AlainI18NService接口，这个接口是这样定义的：
``` typescript
export interface AlainI18NService {
    [key: string]: any;
    use(lang: string, firstLoad: boolean): void;
    getLangs(): any[];
    fanyi(key: string): any;
}
```
2. 构造方法中调用``ngx-translate``的``addLangs``方法来增加可选语言。``use``方法内设置了ng-zorro的国际化，如果是初次加载时，需要重新刷新页面，也就是重定向到``/``，``use``方法的最后，调用了``ngx-translate``的``use``方法。这样，就可以使用``assets/i18n``中的文件，然后利用模板内的``translate``标识符进行翻译。
```
export function HttpLoaderFactory(http: HttpClient) {
    return new TranslateHttpLoader(http, `assets/i18n/`, '.json');
}.

```

接下来，core模块实现了http拦截器，通过对``HttpInterceptor``接口的实现，将错误全部导入``handleData``方法进行处理，然后根据错误代码导向不同的页面。

最后，core模块制作了一个初始化器，用来加载app内必要的数据。然后设置应用信息，用户信息，初始化菜单等，这期间，调用了@delon theme中的几个服务，例如：``MenuService,SettingsService，TitleService``，这几个服务将会贯穿整个app的始终。

## layout模块

layout模块制定了页面布局，也就是以下几个部分：

-  header 
-  sidebar
-  passport
-  default: 页面的标准布局

其中，header，sidebar的每一个部分都是一个独立的组件，这两个模块的详细解析放在专门章节解读。

passport模块定义了路由插槽，login,register两个模块可以在passport模块上来回切换。

default页面的布局如下：
``` html
<div class="wrapper">
    <div class="router-progress-bar" *ngIf="isFetching"></div>
    <app-header class="header"></app-header>
    <app-sidebar class="aside"></app-sidebar>
    <section class="content">
        <!-- 引用自@delon/abc/reuse-tab -->
        <reuse-tab></reuse-tab>
        <router-outlet></router-outlet>
    </section>
</div>
```

default模块内订阅了router事件，当发生错误时停止``router-progress-bar``的滚动，100毫秒后隐藏``progress-bar``，它的css在@delon/theme/styles/app/router-progress-bar.less中。

## routes模块

routes模块中定义了整个app的路由表和对应的组件，事实上把路由表与组件分开会让src的结构更加清晰。

## shared模块

将常用模块包装后又导出。跟正常的shared模块的用途一致。