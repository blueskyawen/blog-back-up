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

### 基本应用
##### （1）获取数据
**基本格式**

    import {Observable} from "rxjs/Observable";
    import {HttpClient} from "@angular/common/http";

    http.get(url).pipe(
        map(...),
        catchError(...)
    ).subscribe(
        data => {...},
        err => {
          console.log('Something went wrong!');
        }
    });

比如：

    this.http.get('/api/items').pipe(
        map(res => res.items || []),
    ).subscribe(data => {
        this.results = data['results'];
    });

**限制强类型参数**

    http.get<Config>(url).subscribe((data : Config) => {
      this.results = data.results;
    });

    或

    http.get<Config>(url).subscribe((data : Config) => {
      this.result = {...data};
    });

比如：

    interface ItemsResponse {
      results: string[];
    }

    this.http.get<ItemsResponse>('/api/items').pipe(
        map((res : ItemsResponse) => res.results || []),
    ).subscribe(data => {
        this.results = data;
    });

    或

    getItems(url) : Observable<ItemsResponse> {
        return this.http.get<ItemsResponse>('/api/items').pipe(
            map((res : ItemsResponse) => res.results || []);
    }

**读取完整的响应体**
读取完整的响应体，包括特殊的响应头或状态码

    //get返回对象Observable<HttpResponse<Config>>
    http.get<Config>(url, {observe: 'response'})
      .subscribe(resp => {
        console.log(resp.headers.get('X-Custom-Header'));
        console.log(resp.body.someField);
    });

比如：

    this.http.get<ItemsResponse>('/api/items'，{observe: 'response'})
        .subscribe(resp => {
            console.log(resp.headers.get('X-Custom-Header'));
            this.results = resp.body;
    });

**错误处理和失败重试**

    handleError(error: HttpErrorResponse) {
        ...
        return of([]);
    }

比如：

    private handleError(error: HttpErrorResponse) {
        if (error.error instanceof ErrorEvent) {
          console.error('An error occurred:', error.error.message);
        } else {
          console.error(`Backend returned code ${error.status}, ` +`body was: ${error.error}`);
        }
        return throwError('Something bad happened！');
    };

    this.http.get('/api/items').pipe(
        retry(3),
        catchError(handleError)
    ).subscribe(data => {
        this.results = data['results'];
    },error => {
        console.error('error is '+error);
    });

**请求非 JSON 数据**

     this.http.get(url, {responseType: 'text'})
        .pipe(
          tap(
            data => this.log(filename, data),
            error => this.logError(filename, error)
          )
        );

比如：

     this.http.get('assets/not-json.txt', {responseType: 'text'})
        .pipe(
          tap(
            data => this.log(filename, data),
            error => this.logError(filename, error)
          )
        ).subscribe(data => {
            this.content = data;
        });


##### （2）添加数据
**基本格式**

    http.post(url, body, {headers: new HttpHeaders()})
        .pipe(
            map(...),
            catchError(...)
        ).subscribe(...);

比如：

    const body = {name: 'Brad'};
    const url = '/api/developers/add';
    http.post(url, body, {headers: new HttpHeaders()})
        .pipe(
            catchError(this.handeError)
        ).subscribe(data => {
            this.data = data
        });

**添加请求头配置**

    import {HttpHeaders} from "@angular/common/http";

    http.post(url, body,
        {headers: new HttpHeaders().set(‘Authorization’, ‘my-auth-token’)})
        .pipe(
            catchError(this.handeError)
        ).subscribe(...);

你无法直接修改前述配置对象中的现有头，因为HttpHeaders实例是不可变的，如果需要增加或修改头配置，则clone一份即可

    headers =
      new HttpHeaders().set('AuthToken', 'a123456');

**携带附加参数**

    import {HttpParams} from "@angular/common/http";

    http.post(url, body,
        {params: new HttpParams().set('id', '3')})
        .subscribe(...);

效果同下

    http.post(url+'?id=3', body)
      .subscribe();

同样的，HttpParams实例是不可变的，如果需要增加或修改配置，需要clone一份

    params =
      new HttpParams().set('name', 'a123456');

##### （3）修改数据
与post类似，只是换成了put而已

    http.put(url, body,
        {headers: new HttpHeaders().set(‘Authorization’, ‘my-auth-token’)})
        .pipe(
            catchError(this.handeError)
        ).subscribe(...);

##### （4）删除数据

    http.delete(url, id)
      .pipe(
            catchError(this.handeError)
        ).subscribe();

> 注意：delete操作虽然返回没什么可处理的，但是subscribe必须要有，否则不会执行

参考示例：
[config](http://blueskyawen.com/angular-work-cook/main/advance/http/config)
[TextLoad](http://blueskyawen.com/angular-work-cook/main/advance/http/textloader)
