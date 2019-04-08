---
title: Angular 利用ng-zorro实现全局http错误拦截与提示
date: 2016-5-23
categories:
- 前端
- angular
tags:
- 前端
- angular
- ng-zorro
---
# 拦截http错误

http全局错误的拦截可以采用先扩展``Http``类，重写请求方法后发出错误，之后实现``ErrorHandler``类来实现。
下面是部分关键代码：
<!-- more -->
``` typescript
export class InterceptedHttp extends Http {

    constructor(
        backend: ConnectionBackend, 
        defaultOptions: RequestOptions) {
        super(backend, defaultOptions);
    }

    request(url: string | Request, options?: RequestOptionsArgs): Observable<Response> {
        // 在此处可以写一些拦截请求的操作
        this.updateUrl(url);
        return super.request(url, options);
    }

    get(url: string, options?: RequestOptionsArgs): Observable<Response> {
        return super.get(url, this.getRequestOptionArgs(options));
    }
    
    ....
    // 改写请求url,加上req
    private updateUrl(req: string) {
        return  "/api/" + req;
    }
    // 改写请求头部
    private getRequestOptionArgs(options?: RequestOptionsArgs) : RequestOptionsArgs {
        if (options == null) {
            options = new RequestOptions();
        }
        if (options.headers == null) {
            options.headers = new Headers();
        }
        options.headers.append('Content-Type', 'application/json');

        return options;
    }
}
```
以上是一个简单的http请求拦截器，它可以改写请求参数，改写请求头部，还有一种写法是实现``HttpInterceptor``接口（angular5新添加)。
``` typescript
export class MyHttpInterceptor implements HttpInterceptor {
constructor() { }

intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {

console.log("intercepted request ... ");

// Clone the request to add the new header.
const authReq = req.clone({ headers: req.headers.set("headerName", "headerValue")});

console.log("Sending request with new header now ...");

//send the newly created request
return next.handle(authReq)
.catch((error, caught) => {
    //intercept the respons error and displace it to the console
    console.log("Error Occurred");
    console.log(error);
    //return the error to the method that called it
    return Observable.throw(error);
}) as any;
}
}
```
接下来实现ErrorHandler类，它可以捕捉各种类型的错误，包括Http请求错误。
``` typescript
export class CustomErrorHandler implements ErrorHandler {

    constructor( @Inject(NotificationService) private notificationService: NotificationService) {
    }

    handleError(error: any): void {
        let msg = this.httpErrorHandler(error.rejection || error);
    }

    httpErrorHandler(err): string {
        let message = '';
        if (err.status) {
            switch (err.status) {
                case 502:
                case 500:
                    message = `服务器故障 : \n详细信息 : ${err.statusText}:${err.json().message}`;
                    break;
                case 401:
                    message = `${err.json().message}`
                    break;
                default:
                    break;
            }
        } else {
            message = err.message;
        }
        return message;
    }
```
# 发送错误消息
捕捉到错误信息以后，需要通知某个服务来显示消息，利用nzMessageService将其显示出来，nzMessageService显示时需要视图载体，这里采用AppComponent作为载体。

为ErrorHandler添加错误通知功能
``` typescript
    handleError(error: any): void {
        let msg = this.httpErrorHandler(error.rejection || error);
        this.notificationService.error(msg);
    }
```
notificationService必须类似于EventEmitter一样，收到消息后自动发射到接受方，所以可以采用Subject来作为Message的主体，让Message可以被订阅，也可以被按需求发射。

Message的模型如下:

``` typescript
export class Message{
    type: string;
    message: string;
    constructor(type, message){
        this.type = type;
        this.message = message;
    }
}

export class NotificationService {
    public message: Subject<Message> = new Subject<Message>();

    constructor() {
    }

    error(message: string): void {
        this.message.next(new Message('error', message));
    }
}
```
之后，只要在AppComponent中订阅Message，就可以在前台进行提示了！
``` typescript
export class AppComponent implements OnInit {
  title = 'app';
  constructor(private notification: NotificationService,
    private nzMessageService: NzMessageService) {
  }

  ngOnInit(){
    this.notification.message.distinctUntilChanged().subscribe((msg)=>{
      this.nzMessageService.create(msg.type, msg.message);
    });
  }
}
```
最后，注册以上这几个服务到AppModule中。
``` typescript
export function httpFactory(xhrBackend: XHRBackend, requestOptions: RequestOptions): Http {
  return new InterceptedHttp(xhrBackend, requestOptions);
}

providers: [
    {
      provide: Http,
      useFactory: httpFactory,
      deps: [XHRBackend, RequestOptions]
    },
    NotificationService, // added
    { provide: ErrorHandler, useClass: CustomErrorHandler },
    NzMessageService,
    { provide: NZ_MESSAGE_CONFIG, useValue: { nzDuration: 7000 } }
  ],
```