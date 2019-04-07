---
title: Angular5 jwt验证
date: 2016-5-21
categories:
- 前端
- angular
tags:
- 前端
- angular
---
首先写一个简单的bootstrap的登陆表单。
新建loginComponent，然后在hmtl里写:
``` html
<div class="col-md-6 col-md-offset-3">
  <h2>Login</h2>
  <form name="form" (ngSubmit)="f.form.valid && login()" #f="ngForm" novalidate>
      <div class="form-group" [ngClass]="{ 'has-error': f.submitted && !username.valid }">
          <label for="username">Username</label>
          <input type="text" class="form-control" name="username" [(ngModel)]="user.username" #username="ngModel" required />
          <div *ngIf="f.submitted && !username.valid" class="help-block">Username is required</div>
      </div>
      <div class="form-group" [ngClass]="{ 'has-error': f.submitted && !password.valid }">
          <label for="password">Password</label>
          <input type="password" class="form-control" name="password" [(ngModel)]="user.password" #password="ngModel" required />
          <div *ngIf="f.submitted && !password.valid" class="help-block">Password is required</div>
      </div>
      <div class="form-group">
          <button [disabled]="loading" class="btn btn-primary">Login</button>
          <i class="fa fa-spinner fa-spin" *ngIf="loading"></i>
      </div>
  </form>
</div>
```
在loginComponent内写：
``` typescript
export class LoginComponent implements OnInit {
    user: User = new User();
    loading = false;
    error = '';

    constructor(
        private router: Router,
        private authenticationService: AuthenticationService) { }

    ngOnInit() {
        // reset login status
        this.authenticationService.logout();
    }

    login() {
        this.loading = true;
        this.authenticationService.login(this.user)
            .subscribe(result => {
                if (result === true) {
                    this.router.navigate(['/']);
                } else {
                    this.error = 'Username or password is incorrect';
                    this.loading = false;
                }
            },error=>{
                this.loading = false;
            });
    }
}
```
UserModel的代码：
``` typescript
export class User{
    id: number;
    username: string;
    password: string;
}
```
login后，要根据用户名密码来在后台签名，签名结果token返回前台，前台接收到token后，存储到localstorage中，然后以后每次请求时，都要带上token作为头部。
``` typescript
const User = require('../models/user.model');
const jwt = require('jsonwebtoken');
const secret = require('../secret');
async function login(ctx) {
  const { query } = ctx.request
  let user = await User.find({where: {id: query.id}});
  if (query.password === user.password) {
    ctx.status = 200
    ctx.body = {
      message: '登录成功',
      user: user,
      token: jwt.sign({
        data: user,
        exp: Math.floor(Date.now() / 1000) + (60 * 60), // 60 seconds * 60 minutes = 1 hour
      }, secret),
    }
  } else {
    let err = new Error('Password Error');
    err.status = 401;
    err.message = `密码错误`;
    throw err;
  }
}

module.exports = { login };
```
koa-jwt添加进来以后，引入公钥，然后排除login api：
``` 
jwt({ secret: secret }).unless({ path: [/^\/api\/login/] })
```
这样，当前台发出请求时，就会得到签名过的token，为了统一处理，写一个共通处理的httpinterceptor，这里面用到了``ngx-notify``。
``` typescript
export class DefaultHttpInterceptor implements HttpInterceptor {

    constructor(private auth: AuthenticationService, private notifyService: NotifyService) {
    }

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpSentEvent | HttpHeaderResponse | HttpProgressEvent | HttpResponse<any> | HttpUserEvent<any>> {

        let url = req.url;
        if (!url.startsWith('https://') && !url.startsWith('http://')) {
            url = environment.api + url;
        }
        let options = { url: url };
        if (!url.includes('login')) {
            options['setHeaders'] = {
                Authorization: `Bearer ${this.auth.getToken()}`
            }
        }
        const newReq = req.clone(options);
        const started = Date.now();
        return next.handle(newReq).pipe(
            tap(event => {
              if (event instanceof HttpResponse) {
                const elapsed = Date.now() - started;
                console.log(`Request for ${req.urlWithParams} took ${elapsed} ms.`);
              }
            }, error => {
                switch (error.status) {
                    case 401:
                        // 权限处理
                        this.notifyService.error('401', `错误代码为：${error.message}`);
                        break;
                    case 404:
                        this.notifyService.error('404', `API不存在`);
                        break;
                    case 500:
                        this.notifyService.error('500', error.message);
                        break;
                }
            }));
    }
}
```

过程中遇到的坑：
1. httpinterceptor只支持httpClient而不支持http。
2. koa-jwt只支持自动校验，不支持sign。
