---
title: "All Error Handler Class"
comments: true
categories: angular
tags: [error, module]
date: 2019-07-01 17:37:00 +0900
---

Catch All Angular Errors with a Custom Error Handler

## 1. How to Use
Leverage Angular's built-in ErrorHandler to catch all errors globally. By implementing the handleError method, you can centralize your error-handling logic in one place. This approach applies to both the Module-based and Standalone implementations of Angular apps.

## 2. Code
### Module-Based Implementation
If your Angular app is not using the Standalone API (i.e., you're using the traditional Module-based approach), you can implement the custom error handler as follows:

The NgModule refers to the root module of your application (e.g., AppModule). Add the providers array to the module to register your custom error handler.

```ts
class MyErrorHandler implements ErrorHandler {
  handleError(error) {
    // Handle the error globally
  }
}

@NgModule({
  providers: [{ provide: ErrorHandler, useClass: MyErrorHandler }]
})
class AppModule {}
```

### Standalone Implementation
In a Standalone Angular app, you can create a custom error handler by registering the provider in the bootstrapApplication method or directly within the component.

#### 1. Global Error Handler
To handle errors globally in a Standalone app, use the bootstrapApplication method:

```ts
import { ErrorHandler, Injectable } from '@angular/core';
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

@Injectable()
class MyErrorHandler implements ErrorHandler {
  handleError(error: any): void {
    console.error('Global Error:', error);
  }
}

bootstrapApplication(AppComponent, {
  providers: [
    { provide: ErrorHandler, useClass: MyErrorHandler },
  ]
});
```

#### 2. Local Error Handler
If you need to register the error handler locally within a Standalone component, do the following:

```ts
import { Component, ErrorHandler } from '@angular/core';

@Component({
  standalone: true,
  providers: [
    { provide: ErrorHandler, useClass: MyErrorHandler }
  ],
  template: ``
})
export class MyStandaloneComponent {
  // Component-specific logic
}
```

And that's it! Whether you’re using a Module-based or Standalone approach, you can now handle all Angular errors with ease.

## Link
[한국어(Korean) Page](https://velog.io/write?id=3b38cdfa-61ad-4ed3-a28e-cec6740d357e)
<br/>