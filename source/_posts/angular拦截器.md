---
title: angular拦截器
date: 2018-11-03 21:33:04
tags: 
- angular
categories: 
- angular
---

# angular拦截器

##### 需求是在请求401的时候跳转到登陆页

##### 1.angular中http请求，依赖@angular/common/http模块，将HttpInterceptor，HttpRequest，HttpEvent，HttpHandler等对象引入

```javascript
import { Injectable } from '@angular/core';
import {
  HttpEvent, HttpInterceptor, HttpHandler, HttpRequest
} from '@angular/common/http';

import { Observable } from 'rxjs';

@Injectable()
export class NoopInterceptor implements HttpInterceptor {

  intercept(req: HttpRequest<any>, next: HttpHandler):
    Observable<HttpEvent<any>> {
    return next.handle(req);
  }
}
```

##### 2.了解一下HttpInterceptor，HttpRequest，HttpEvent，HttpHandler

```typescript
interface HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>>
}

//Parameters
// req	HttpRequest	
//next	HttpHandler

//Returns
//Observable<HttpEvent<any>>

//通常拦截器将在返回next.handle()之前，发出请求，拦截器也可以通过在next.handle()返回到流，使用RXJS的其他操作符。
```



```typescript
class HttpRequest<T> {
  constructor(method: string, url: string, third?: T | { headers?: HttpHeaders; reportProgress?: boolean; params?: HttpParams; responseType?: "array..., fourth?: { headers?: HttpHeaders; reportProgress?: boolean; params?: HttpParams; responseType?: "arraybuff...)
  body: T | null
  headers: HttpHeaders
  reportProgress: boolean
  withCredentials: boolean
  responseType: 'arraybuffer' | 'blob' | 'json' | 'text'
  method: string
  params: HttpParams
  urlWithParams: string
  url: string
  serializeBody(): ArrayBuffer | Blob | FormData | string | null
  detectContentTypeHeader(): string | null
  clone(update: { headers?: HttpHeaders; reportProgress?: boolean; params?: HttpParams; responseType?: "arraybuff... = {}): HttpRequest<any>
}
//HttpRequest代表一个发送出去的请求，包括url，method，headers，body等等。
```



```typescript
type HttpEvent<T> = HttpSentEvent | HttpHeaderResponse | HttpResponse<T> | HttpProgressEvent | HttpUserEvent<T>;
```



```typescript
abstract class HttpHandler {
  abstract handle(req: HttpRequest<any>): Observable<HttpEvent<any>>
}
//Rarameters
// req	HttpRequest	

//Returns
//Observable<HttpEvent<any>>

//将http请求转成http对象流。
```

##### 3.实现在401的时候跳转至登陆页

```typescript
import { Injectable } from '@angular/core';
import {
  HttpEvent, HttpInterceptor, HttpHandler, HttpRequest
} from '@angular/common/http';

import { Observable, throwError } from 'rxjs';
import {  catchError } from 'rxjs/operators';
import { ErrorObservable } from 'rxjs/observable/ErrorObservable';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler):
    Observable<HttpEvent<any>> {
    const clonedRequest = req.clone({
      setHeaders: {
        authorization: `这里放token`
      }
    });
    return next.handle(clonedRequest).pipe(
      catchError(function(error){
        if(error.status === 401){
            //跳转登陆页
        }
        return ErrorObservable.create(error);
      })
    );
  }
}
```









