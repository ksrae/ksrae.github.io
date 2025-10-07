---
title: "missing the last word while input 2-byte letters"
date: 2020-09-03 17:31:00 +0900
comments: true
categories: angular
tags: [input, form]
---



## Cause

A common issue across various applications is the phenomenon where the last character is dropped when entering Korean or other double-byte characters in an input field.  This often occurs due to Input Method Editor (IME) composition issues.

While this can be temporarily resolved by entering a non-character key (such as an arrow key or document key), this workaround is inconvenient for users.

Angular addresses this problem by providing `COMPOSITION_BUFFER_MODE` within the `@angular/forms` module.

## Solution

Set the `COMPOSITION_BUFFER_MODE` within the initial execution module (typically `app.module.ts`).

### Under Angular 17
```tsx
import { COMPOSITION_BUFFER_MODE } from '@angular/forms';
...

@NgModule({
    providers: [
        { provide: COMPOSITION_BUFFER_MODE, useValue: false }
    ]
})

...
```

### Angular 17+
```tsx
// src/app/app.config.ts

import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { COMPOSITION_BUFFER_MODE } from '@angular/forms';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),

    { provide: COMPOSITION_BUFFER_MODE, useValue: false }
  ]
};
```

This configuration is briefly described on the official Angular website, as follows:

```
Provide this token to control if form directives buffer IME input until the "compositionend" event occurs.
```

Disabling the composition buffer mode resolves the lost character issue.

## References

- [Input Text Cut-off Issue When Typing Korean](https://gogomalibu.tistory.com/151?category=851911)
- [COMPOSITION_BUFFER_MODE](https://angular.io/api/forms/COMPOSITION_BUFFER_MODE)