---
title: "XSRF 방어하기 (XSRF Protection)"
date: 2020-10-26 14:00:00 +0900
comments: true
categories: angular
tags: [xsrf, csrf, protection]
---


## 환경
Angular 10

  
## 원인
(서버에서 xsrf 또는 csrf를 허용한 경우)<br/>
Angular 6 이하에서는 http 전송 시 csrf 쿠키값을 헤더에 넣기 위해 HttpInterceptor를 사용하였는데, <br/>
여기에 csrf의 값이 담긴 쿠키값을 가져와 헤더를 생성하여야 하는 불편함이 있었습니다.<br/>

```tsx
@Injectable({ providedIn: 'root' })
export class ApiInterceptor implements HttpInterceptor {
  constructor(private cookieService: CookieService) { }

  intercept(req: HttpRequest<any>, handler: HttpHandler): Observable<HttpEvent<any>> {
    const headers = {
      'My-Xsrf-Header': this.cookieService.get('My-Xsrf-Cookie'),
    };

    return handler.handle(
      req.clone({ setHeaders: headers, withCredentials: true })
    );
  }
}
```

Angular 7 버전부터 HttpClientXsrfModule을 지원하여, 이를 보다 손쉽게 정리할 수 있습니다.<br/>
모듈에서 직접 설정할 수 있기 때문에 하드코딩이 상위로 올라와 관리 포인트를 줄일 수 있으며, 직관적인 확인이 가능합니다. <br/> 

```tsx
import { HttpClientModule, HttpClientXsrfModule } from '@angular/common/http';

imports: [
  HttpClientModule,
  HttpClientXsrfModule.withOptions({
    cookieName: 'My-Xsrf-Cookie',
    headerName: 'My-Xsrf-Header',
  }),
],
```

## 에러

이렇게 설정하였는데도 막상 실행해보면 에러가 발생한다면, <br/>
이는 인증을 이용한 요청방식을 위해 서버에서는 아래와 같이 Credentials 값을 true로 요구하고 있기 때문입니다.<br/>

```
response.setHeader("Access-Control-Allow-Credentials", "true")
```



## 해결
Angular에서 http request 마다 또는 Interceptor에서 withCredential 값을 true로 설정해주어야 합니다.<br/>


```tsx
@Injectable()
export class HttpConfigInterceptor implements HttpInterceptor {
  constructor() {}

  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    request = request.clone({
      withCredentials: true
    });

    return next.handle(request);
  }
}
```



## 참고 사이트
- [XSRF protection](https://angular.io/guide/http#security-xsrf-protection)
- [HttpRequest](https://angular.io/api/common/http/HttpRequest)