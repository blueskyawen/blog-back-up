---
title: angular-http拦截
date: 2019-01-29 21:27:00
tags: angular
categories: 前端
comments: true
toc: true
---

在客户端和服务器进行http交互，人们能够对过程进行一定程度的干预，也希望这么做，比如：对http的响应进行修缮，或者对请求头设置维护等。  
这很有用，XSRF攻击的通用防范手段就是通过增加头部的token来和服务器进行访问验证的  
实现这种目的最简单的方式就是在每次http请求的前后增加预处理，比如：

    const httpOptions = {
      headers: new HttpHeaders({
        'Content-Type':  'application/json',
        'Authorization': 'my-auth-token'
      })
    };
    this.http.post(url,data,httpOptions).pipe(
        map(...) //响应处理
    );

<!--more-->
这个对于有特殊处理方式的请求可以的，但是对于那些通用的处理太繁琐了，维护困难，于是边有了更加通用的处理手段：代理服务和http拦截器  

# 代理服务  
基于设计里的代理模式，这是包装原有http服务的一种手段，通过提供一个新服务来处理http请求和响应结果，接口和原http服务一致  
下面是典型的代理http服务

    import { Injectable } from '@angular/core';
    import { Headers, Http, ResponseContentType,HttpErrorResponse } from '@angular/http';
    import { AuthService } from './AuthService.service';
    import { Router } from '@angular/router';
    import { Observable } from 'rxjs/Observable';
    import './rxjs-operators';
    
    @Injectable({
        providedIn: 'root'
    })
    export class HttpApiService {
        token: any;
        headers: any = null;
        lang: any;
    
        constructor(private http: Http,
                private authService: AuthService,
                private router: Router) {
            this.token = this.authService.getToken();
            this.lang = this.authService.getLang();
            this.headers = new Headers({
                                'token':this.token,
                                'language':this.lang
                            });
        }
    
        get(url: any, params?: any) {
            let paramOption = params ? new HttpParams(params) : {};
            return this.http.get(url,paramOption).pipe(
                catchError(this.handleError)
            );
        }
    
        post(url: any, data: any, params?: any) {
            let paramOption = params ? new HttpParams(params) : {};
            return this.http.post(url,data,paramOption).pipe(
                catchError(this.handleError)
            );
        }
    
        delete(url: any) {
            return this.http.delete(url).pipe(
                catchError(this.handleError)
            );
        }
    
        put(url: any, data: any, params?: any) {
            let paramOption = params ? new HttpParams(params) : {};
            return this.http.put(url,data,paramOption).pipe(
                catchError(this.handleError)
            );
        }
    
        handleError(error: HttpErrorResponse,result ?: any) {
          let errorMsg : string;
          if (error.error instanceof ErrorEvent) {
            errorMsg = 'An error occurred:' + error.error.message;
          } else {
            switch (error.status) {
                case 500:
                    errorMsg = '内部服务错误,错误信息:' + error.error;
                    break;
                case 401:
                    errorMsg = '无访问权限,错误信息:' + error.error;
                    break;
                case 404:
                    errorMsg = '页面不存在,错误信息:' + error.error;
                    break;
                default:
                    errorMsg = '其他错误,错误信息:' + error.error;
            }
          }
          return result ? Observable.of(result) : 
                    Observable.throwError(errorMsg);
        }
    
    }

使用http代理服务

    import { HttpApiService as Http } from '../../apiResource.service';
    
    constructor(private http : Http) {}
    
    getItems() {
        this.http.get(url).subscribe(res => {
            this.items = res;
        },error => {
            this.error = error;
        });
    }
    
    addItem() {
        this.http.post(url,this.data).subscribe(res => {
            this.items = res;
        },error => {
            this.error = error;
        });
    }
    
    deleteItem() {
        this.http.delete(url).subscribe();
    }

# 拦截器  
HTTP 拦截机制是angular的主要特性之一，使用这种拦截机制，你可以声明一些拦截器，用它们监视和转换从应用发送到服务器的 HTTP 请求，还可以用监视和转换从服务器返回到本应用的那些响应   
所以，利用好这个特性，便可以使用原始http的情况下也能进行预处理和响应转换了  

## 编写拦截器
拦截器的实现很简单，就要实现一个HttpInterceptor接口中的intercept() 方法即可  

    import { Injectable } from '@angular/core';
    import {HttpEvent, HttpInterceptor,HttpHandler,
            HttpRequest,HttpResponse} from '@angular/common/http';
    
    @Injectable()
    export class LoggingInterceptor implements HttpInterceptor {
        constructor() {}
        
        intercept(req: HttpRequest<any>, 
                  next: HttpHandler):Observable<HttpEvent<any>> {
            ...doing something before send req...
            return next.handle(req).pipe(
                    ..doing something width response..
                );
        }
    }

可以看到：

- 拦截器一般以Interceptor结尾命名，继承接口HttpInterceptor，实现intercept方法
- 拦截器在拿到req之后可以做一些预处理，然后再交给下一个拦截器或后端服务器
- 使用next.handle()方法将请求交给下一个拦截器或服务器，next指的就是下个拦截器对象，handle方法的最后一个next是后端服务器
- 拦截器在得到响应之后可以做数据处理，然后再把响应信息交给用户

下面是实现上面代理服务功能的一些拦截器：

**添加请求头**  

    import { Injectable } from '@angular/core';
    import {HttpEvent, HttpInterceptor, HttpHandler, 
            HttpRequest} from '@angular/common/http';
    
    @Injectable()
    export class AuthInterceptor implements HttpInterceptor {
        constructor(private authService: AuthService) {}
    
        intercept(req: HttpRequest<any>,
                  next: HttpHandler): Observable<HttpEvent<any>> {
            let token = this.authService.getToken();
            let lang = this.authService.getLang();
            let tmpHeaders = req.headers.set('token',token)
                                .set('language',this.lang);
            let authReq = req.clone({ headers: tmpHeaders});
            return next.handle(req);
        }
    }
    
> 注意: Headers是不可变的，不能直接修改它，可复制修改后再赋值

**修改请求体**  

    import { Injectable } from '@angular/core';
    import {HttpEvent, HttpInterceptor, HttpHandler, 
            HttpRequest} from '@angular/common/http';

    @Injectable()
    export class ReqModifyInterceptor implements HttpInterceptor {
        intercept(req: HttpRequest<any>,
                  next: HttpHandler): Observable<HttpEvent<any>> {
            const secureReq = req.clone({
                url: req.url.replace('http://', 'https://')
            });
            return next.handle(secureReq);
        }
    }

>  HttpReques和HttpResponse是不可变的，不能直接修改它，可复制修改后再赋值

**耗时记录** 

    import { Injectable } from '@angular/core';
    import {HttpEvent, HttpInterceptor, HttpHandler, 
            HttpRequest, HttpResponse} from '@angular/common/http';
    import { finalize, tap } from 'rxjs/operators';
    import { Observable } from 'rxjs';

    @Injectable()
    export class TimingInterceptor implements HttpInterceptor {
        intercept(req: HttpRequest<any>,
                  next: HttpHandler): Observable<HttpEvent<any>> {
            let started = Date.now();
            let state: string;
            return next.handle(req).pipe(
                tap(event => ok = event instanceof HttpResponse ?
                                'succeeded' : '',
                    error => ok = 'failed'),
                finalize(() => {
                    let elapsed = Date.now() - started;
                    const msg = `${req.method} "${req.urlWithParams}"
                        ${ok} in ${elapsed} ms.`;
                    console.log(msg);
                })
            );
        }
    }
    
**错误捕获**   

    import { Injectable } from '@angular/core';
    import {HttpEvent, HttpInterceptor, HttpHandler, HttpErrorResponse,
            HttpRequest, HttpResponse} from '@angular/common/http';
    import { finalize, tap } from 'rxjs/operators';
    import { Observable } from 'rxjs';

    @Injectable()
    export class ErrorCatchInterceptor implements HttpInterceptor {
        let errorMsg : string;
        intercept(req: HttpRequest<any>,
                  next: HttpHandler): Observable<HttpEvent<any>> {
            return next.handle(req).pipe(
                catchError((error: HttpErrorResponse) => {
                    if (error.error instanceof ErrorEvent) {
                        errorMsg = 'An error occurred:' + error.error.message;
                    } else {
                        switch (error.status) {
                            case 500:
                                errorMsg = '内部服务错误,错误信息:' + error.error;
                                break;
                            case 401:
                                errorMsg = '无访问权限,错误信息:' + error.error;
                                break;
                            case 404:
                                errorMsg = '页面不存在,错误信息:' + error.error;
                                break;
                            default:
                                errorMsg = '其他错误,错误信息:' + error.error;
                        }
                      }
                      return result ? Observable.of(result) : 
                                Observable.throwError(errorMsg); 
                })
            );
        }
    }
    
## 注册拦截器
拦截器也是服务，需要将其注册到注入器以后才能使用，而且由于他们是和http配和使用的，所以必须在httpClient服务注册的同一级提供这些拦截器服务，否则不生效。  
比如，当httpClient注入到根注入器后，拦截器也应该注入到根注入器，下面都注入根  

    //interceptor-index.ts
    import { HTTP_INTERCEPTORS } from '@angular/common/http';
    import { AuthInterceptor } from './auth-interceptor';
    import { ReqModifyInterceptor } from './req-modify-interceptor';
    import { TimingInterceptor } from './timing-interceptor';
    import { ErrorCatchInterceptor } from './error-catch-interceptor';
    
    export const httpInterceptorProviders = [
        { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
        { provide: HTTP_INTERCEPTORS, useClass: ReqModifyInterceptor, multi: true },
        { provide: HTTP_INTERCEPTORS, useClass: TimingInterceptor, multi: true },
        { provide: HTTP_INTERCEPTORS, useClass: ErrorCatchInterceptor, multi: true }
    ];

    //core.module.ts
    import { NgModule } from '@angular/core';
    import { CommonModule } from '@angular/common';
    import { HttpClientModule }    from '@angular/common/http';
    import { httpInterceptorProviders } from './interceptor-index';

    @NgModule({
      declarations: [],
      imports: [
        CommonModule
      ],
      exports: [HttpClientModule],
      providers: [httpInterceptorProviders]
    })
    export class CoreModule { }

可以看到：

- 拦截器是以数组形式提供的，multi: true代表一个多重提供商的令牌，即一个令牌注入注入多数组
- 多个拦截器按顺序从上往下进行注册，一次请求过程中拦截器的处理顺序也是从上往下传递的，响应的方向正好相反，如下示意图

![hexo-interceptor](/images/http-interceptor.jpg)

## 示例

[头配置](http://blueskyawen.com/angular-work-cook/main/advance/http/config)  
[Txt文档下载](http://blueskyawen.com/angular-work-cook/main/advance/http/textloader)  
[文件上传](http://blueskyawen.com/angular-work-cook/main/advance/http/uploader)  
[npm包搜索](http://blueskyawen.com/angular-work-cook/main/advance/http/npmsearch)

