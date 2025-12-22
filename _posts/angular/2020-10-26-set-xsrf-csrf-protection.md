---
title: "XSRF Protection"
date: 2020-10-26 14:00:00 +0900
comments: true
categories: angular
tags: [xsrf, csrf, protection]
---

[한국어(Korean) Page](https://velog.io/@ksrae/XSRF-%EB%B0%A9%EC%96%B4%ED%95%98%EA%B8%B0https://velog.io/@ksrae/ngZoneEventCoalescing%EC%9C%BC%EB%A1%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%B2%84%EB%B8%94%EB%A7%81-%ED%95%B4%EA%B2%B0)
<br/>

## Cause

(If the server has enabled XSRF or CSRF)<br>
In Angular 6 and earlier, HttpInterceptor was used to include the CSRF cookie value in the header during HTTP transmission. This required the inconvenience of retrieving the cookie value containing the CSRF value and creating a header.<br>

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

From Angular 7 onwards, HttpClientXsrfModule is supported, allowing for easier management. <br>
Because it can be configured directly in the module, hard coding can be reduced, and intuitive confirmation is possible.<br>

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

## Error

If an error occurs even after setting it up like this, <br>
this is because the server requires the Credentials value to be true for authentication request methods, as shown below.<br>

```
response.setHeader("Access-Control-Allow-Credentials", "true")
```

## Solution

In Angular, the withCredential value must be set to true for each HTTP request or in the Interceptor.<br>

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

## Additionally: Solution in Modern Angular (v17+)
Starting with Angular 17, the default is to use Standalone APIs without NgModule. Global HTTP configuration is managed in the app.config.ts file using the provideHttpClient provider function and its accompanying features.

### Solution (Standalone API-based - Angular 17+)
Configure both XSRF settings and the interceptor at once in the app.config.ts file.
- XSRF Configuration: Use the withXsrfConfiguration feature.
- withCredentials Configuration: Use the withInterceptors feature to register a functional interceptor.

```tsx
// src/app/app.config.ts

import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
// 1. Import necessary functions from @angular/common/http
import { provideHttpClient, withXsrfConfiguration, withInterceptors, HttpInterceptorFn } from '@angular/common/http';

import { routes } from './app.routes';

// 2. Define a functional interceptor
const credentialsInterceptor: HttpInterceptorFn = (req, next) => {
  const clonedReq = req.clone({
    withCredentials: true,
  });
  return next(clonedReq);
};

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),

    // 3. Add HttpClient configuration to the providers array
    provideHttpClient(
      // XSRF configuration
      withXsrfConfiguration({
        cookieName: 'My-Xsrf-Cookie',
        headerName: 'My-Xsrf-Header',
      }),
      // Interceptor configuration
      withInterceptors([credentialsInterceptor])
    )
  ]
};
```

## References

- [XSRF protection](https://angular.io/guide/http#security-xsrf-protection)
- [HttpRequest](https://angular.io/api/common/http/HttpRequest)