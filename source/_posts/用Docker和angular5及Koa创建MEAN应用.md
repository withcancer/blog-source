---
title: 用Docker和angular5及Koa创建MEAN应用
date: 2017-12-18
categories:
- 全栈
- MEAN
tags:
- 全栈
- mongodb
- mongoose
- nodejs
- angular
---

首先要确保angualr-cli和docker都已经安装完毕。
## client
创建一个新的angular-cli项目mean-docker/angular-client。

在angular-client目录下，创建一个Dockerfile，并写入如下内容
<!-- more -->
```
FROM node:8
# 创建工作目录
RUN mkdir -p /usr/src/app
# 切换到工作目录
WORKDIR /usr/src/app
# 拷贝package.json到工作目录
COPY package.json /usr/src/app
# 运行npm install
RUN npm install
# 拷贝所有source到工作目录
COPY . /usr/src/app
# 开放4200端口
EXPOSE 4200
# 运行npm start
CMD ["npm", "start"]
```
为了防止node_modules文件夹也被拷贝，可以新建一个.dockeringore文件把它过滤掉。

为了确保client端由docker image提供host，需要修改package.json中npm start一节为：
```
"start": "ng serve -H 0.0.0.0",
```
接下来，build这个镜像：
```
 docker build -t angular-client:dev .
```
bulid好镜像之后，可以使用这个镜像来创建容器：
```
docker run -d --name angular-client -p 4200:4200 angular-client:dev
```
## server
在mean-docker目录下创建一个koa-server目录。执行npm init命令，新建一个npm 项目。安装koa和koa-router等组件。

app.js中写入如下内容：
``` javascript
const compose = require('koa-compose');
const Koa = require('koa');
const app = new Koa();
const router = require('./router');

// x-response-time
async function responseTime (ctx, next){
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  ctx.set('X-Response-Time', `${ms}ms`);
};

// logger
async function logger (ctx, next) {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  console.log(`${ctx.method} ${ctx.url} - ${ms}`);
};

const all = compose([
    responseTime,
    logger(),
    router.routes()
]);

app.use(all);

app.listen(3000);
```
router中写入如下内容：
``` javascript
const router = require('koa-router')();

router.get('/',  (ctx, next) => {
    ctx.body = 'api works';
});

module.exports = router;
```
在koa-server目录下，创建一个Dockerfile，并写入如下内容：
```
FROM node:6
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY package.json /usr/src/app
RUN npm install
COPY . /usr/src/app
EXPOSE 3000
CMD ["npm", "start"]
```
同样的，也需要添加一个.dockeringore来忽略node_modules。
为package.json添加npm start命令
```
"start": "node app.js",
```
执行docker build和docker run
```
docker build -t koa-server:dev .
docker run -d --name koa-server -p 3000:3000 koa-server:dev
```

## mongodb

启动一个mongodb容器：
```
docker run -d --name mongodb -p 27017:27017 mongo
```

## compose
``` yml
services:
  angular:
    build: angular-client
    ports:
      - "4200:4200"
      
  koa: 
    build: koa-server
    ports:
      - "3000:3000"

  database:
    image: mongo
    ports:
      - "27017:27017"
```
至此，三个容器就创建好了，接下来补充程序中的实质性内容：

## server端内容补充
首先安装mongoose，为router添加路由和mongoose组件
``` javascript
// User model
const db = 'mongodb://database/mean-docker';

mongoose.connect(db);

const userSchema = new mongoose.Schema({
    name: String,
    age: Number
});

const User = mongoose.model('User', userSchema);

// ...

router.get('/users',async (ctx, next) => {
    await User.find({}, (err, users) => {
        if (err) {
            ctx.status = 500;
            ctx.body = err;
        }else{
            ctx.status = 200;
            ctx.body = users;
        }
    });
});

/* GET one users. */
router.get('/users/:id',async (ctx, next) => {
    await User.findById(ctx.request.param.id, (err, users) => {
        if (err) {
            ctx.status = 500;
            ctx.body = err;
        }else{
            ctx.status = 200;
            ctx.body = users;
        }
    });
});

/* Create a user. */
router.post('/users',async (ctx, next) => {
    let user = new User({
        name: ctx.request.body.name,
        age: ctx.request.body.age
    });

    await user.save(error => {
        if (error) {
            ctx.status = 500;
            ctx.body = error;
        }else{
            ctx.status = 200;
            ctx.body = {
                message: 'User created successfully'
            };
        }
    });
});
```
接下来，连接mongodb和koa-server，在docker-compose中koa-server一节增加这样一句：
``` yml
links:
      - database
```

## 前端内容补充
在app.component.ts中添加如下内容：
``` typescript
  // ...
  // Link to our api, pointing to localhost
  API = 'http://localhost:3000';

  // Declare empty list of people
  people: any[] = [];

  constructor(private http: Http) {}

  // Angular 2 Life Cycle event when component has been initialized
  ngOnInit() {
    this.getAllPeople();
  }

  // Add one person to the API
  addPerson(name, age) {
    this.http.post(`${this.API}/users`, {name, age})
      .map(res => res.json())
      .subscribe(() => {
        this.getAllPeople();
      })
  }

  // Get all users from the API
  getAllPeople() {
    this.http.get(`${this.API}/users`)
      .map(res => res.json())
      .subscribe(people => {
        console.log(people)
        this.people = people
      })
  }
  // ...
```
在app.component.html中增加如下内容：
``` html
<div class="container">
  <div [style.margin-top.px]="10" class="row">
    <h3>Add new person</h3>
    <form class="form-inline">
      <div class="form-group">
        <label for="name">Name</label>
        <input type="text" class="form-control" id="name" #name>
      </div>
      <div class="form-group">
        <label for="age">Age</label>
        <input type="number" class="form-control" id="age" #age>
      </div>
      <button type="button" (click)="addPerson(name.value, age.value)" class="btn btn-primary">Add person</button>
    </form>
  </div>
  <div [style.margin-top.px]="10" class="row">
    <h3>People</h3>
    <div [style.margin-right.px]="10" class="card card-block col-md-3" *ngFor="let person of people">
      <h4 class="card-title">{{person.name}}  {{person.age}}</h4>
    </div>
  </div>
</div>
```
这样，就可以添加用户，查看用户了。
