---
title: Angular-Http
date: 2017-08-27 12:26:40
tags: Augular
categories: 前端
comments: true
---

前端应用需要通过http协议与后端服务器通讯,现代浏览器支持使用两种不同的 API 发起 HTTP 请求：XMLHttpRequest 接口和 fetch() API。
<!--more-->
@angular/common/http中的HttpClient类(旧版的时Http)，Angular 为应用程序提供了一个简化的 API 来实现 HTTP 功能。它基于浏览器提供的XMLHttpRequest接口。 HttpClient带来的其它优点包括：可测试性、强类型的请求和响应对象、发起请求与接收响应时的拦截器支持，以及更好的、基于可观察（Observable）对象的错误处理机制

### 引入Http模块
在应用根模块引入一次即可，这样可在全应用范围内可用

    import {HttpClientModule} from '@angular/common/http';

    @NgModule({
      imports: [
        BrowserModule,
        HttpClientModule,
      ],
    })
    export class AppModule {}

引入之后，在应用的各个组件和服务里就可以通过依赖注入来使用此服务了

### 基本的应用
**（1）获取数据**
基本格式：

    http.get<ItemsResponse>(url).subscribe(
                 // Successful responses call the first callback.
        data => {...},
                // Errors will call this callback instead:
        err => {
          console.log('Something went wrong!');
        }
      });

比如：

    this.http.get('/api/items').subscribe(data => {
      this.results = data['results'];
    });

限制强类型参数，可直接使用.好获取响应体

    interface ItemsResponse {
      results: string[];
    }
    http.get<ItemsResponse>('/api/items').subscribe(data => {
      this.results = data.results;
    });

读取完整的响应体，包括特殊的响应头或状态码

    http.get<MyJsonData>('/data.json', {observe: 'response'})
      .subscribe(resp => {
        // Here, resp is of type HttpResponse<MyJsonData>.
        // You can inspect its headers:
        console.log(resp.headers.get('X-Custom-Header'));
        // And access the body directly, which is typed as MyJsonData as requested.
        console.log(resp.body.someField);
      });

获取错误信息

    http.get<ItemsResponse>('/api/items').subscribe(
      	data => {...},
        (err: HttpErrorResponse) => {
          if (err.error instanceof Error) {
            // A client-side or network error occurred..
            console.log('An error occurred:', err.error.message);
          } else {
            // The backend returned an unsuccessful response code.
            // The response body may contain clues as to what went wrong,
            console.log(`Backend returned code ${err.status}, body was: ${err.error}`);
          }
        }
      });

请求重试

    import 'rxjs/add/operator/retry';
    http.get<ItemsResponse>('/api/items')
      // Retry this request up to 3 times.
      .retry(3)
      // Any errors after the 3rd retry will fall through to the app.
      .subscribe(...)

请求非 JSON 数据

    http.get('/textfile.txt', {responseType: 'text'})
      // The Observable returned by get() is of type Observable<string>
      // because a text response was specified. There's no need to pass
      // a <string> type parameter to get().
      .subscribe(data => console.log(data));


**（2）设置数据**

    const body = {name: 'Brad'};

    http.post('/api/developers/add', body)
      .subscribe(...);

注意这个subscribe()方法。 所有从HttpClient返回的可观察对象都是冷的（cold），也就是说，它们只是发起请求的蓝图而已。在我们调用subscribe()之前，什么都不会发生，而当我们每次调用subscribe()时，就会独立发起一次请求。下列代码会使用同样的数据发送两次同样的 POST 请求：

    const req = http.post('/api/items/add', body);
    req.subscribe();
    req.subscribe();

请求中添加一个Authorization头

http.post('/api/items/add', body, {
    headers: new HttpHeaders().set('Authorization', 'my-auth-token'),
  })
  .subscribe();

URL携带参数

http.post('/api/items/add', body, {
    params: new HttpParams().set('id', '3'),
  })
  .subscribe();

同下

    http.post('/api/items/add?id=3', body)
      .subscribe();


**（3）删除数据**

    http.delete('/api/developers', id)
      .subscribe(...);

### 拦截器
@angular/common/http的主要特性之一是拦截器，它能声明一些拦截器，拦在应用和后端之间。当应用程序发起一个请求时，拦截器可以在请求被发往服务器之前先转换这个请求。并且在应用看到服务器发回来的响应之前，转换这个响应。这对于处理包括认证和记录日志在内的一系列工作都非常有用

> 可同事拦截请求消息和响应消息

具体可参考文档[HttpClient 库](https://www.angular.cn/guide/http)
