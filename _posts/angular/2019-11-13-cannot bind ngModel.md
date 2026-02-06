---
title: "Cannot bind to 'ngModel' since it isn't a known property of 'input'"
date: 2019-10-17 12:23:00 +0900
comments: true
categories: angular
tags: [error]
---

### Cause

The error "Cannot bind to 'ngModel' since it isn't a known property of 'input'" occurs when using `ngModel` within a form tag without importing the necessary Angular FormsModule.

### Environment

- This issue was observed in Angular version 8.x.

### Solution

To utilize forms properly in Angular, the `FormsModule` must be declared within your module. This module is provided by `@angular/forms`.  Follow these steps to resolve the issue:

- Import `FormsModule` from `@angular/forms`.
- Add `FormsModule` to the `imports` array of your `NgModule`.

```tsx
import { FormsModule } from '@angular/forms';

@NgModule({
    imports: [
        FormsModule
    ]
})
```

This import makes the `ngModel` directive available for use within your Angular templates, enabling two-way data binding with form elements.


### Angular 17 or above

```tsx
// src/app/app.config.ts

import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideFormsModule } from '@angular/forms';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideFormsModule()
  ]
};
```

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Cannot-bind-to-ngModel-since-it-isnt-a-known-property-of-input-%EC%97%90%EB%9F%AC)
<br/>