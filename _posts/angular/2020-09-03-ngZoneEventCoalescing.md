---
title: "ngZoneEventCoalescing for Preventing Event Bubbling"
date: 2020-09-03 17:04:00 +0900
comments: true
categories: angular
tags: [bubbling, changedetection, ngzone]
---

[한국어(Korean) Page](https://velog.io/@ksrae/ngZoneEventCoalescing%EC%9C%BC%EB%A1%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%B2%84%EB%B8%94%EB%A7%81-%ED%95%B4%EA%B2%B0)
<br/>

## Root Cause

JavaScript incorporates event bubbling between parent and child elements. In many scenarios, this event bubbling leads to the handling of unnecessary events, causing performance issues. Similarly, Angular experiences event bubbling, negatively impacting rendering performance.

To address this, change detection is configured. However, in versions prior to Angular 9, even with change detection enabled, timing issues in `ngAfterViewInit` often necessitated encapsulating functions within `setTimeout` for proper execution.

```tsx
ngAfterViewInit(){
    setTimeout(()=> {
        this.changeStatus()
    })
}
```

Angular 9 introduced a more fundamental solution to event bubbling. The `setTimeout` code block above can be replaced with `cdr.detectChanges()` to ensure events are applied correctly without timing conflicts.

```tsx
ngAfterViewInit(){
    this.changeStatus();
    this.cdr.detectChanges();
}
```

## Solution

In `main.ts`, add an option during Angular Module bootstrap configuration:

```tsx
platformBrowserDynamic()
  .bootstrapModule(AppModule, { ngZoneEventCoalescing: true })
```

## Additionally: Solution in Modern Angular (v17+)
Starting with Angular 17, the default architecture is based on Standalone APIs, which do not use NgModule. <br/>
In this environment, bootstrapApplication() in main.ts replaces platformBrowserDynamic().bootstrapModule(), and global application configuration is handled in the app.config.ts file.<br/>
The ngZoneEventCoalescing setting is a configuration option for NgZone. It is now configured using a provider function called provideZoneChangeDetection.<br/>

### Solution (Standalone API-based - Angular 17+)
In your app.config.ts file, add provideZoneChangeDetection to the providers array and pass the option eventCoalescing: true.

```TypeScript
// src/app/app.config.ts

import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),

    // 1. Call the provider function for Zone.js configuration.
    // 2. Enable event coalescing via the options object.
    provideZoneChangeDetection({ eventCoalescing: true })
  ]
};
```

This appConfig is then passed to the bootstrapApplication function in main.ts, applying the configuration to the entire application.


```TypeScript
// src/main.ts (for reference)

import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch((err) => console.error(err));
```

## References

- [how-to-prevent-event-bubbling-in-angular-9-with-setinterval](https://stackoverflow.com/questions/60854223/how-to-prevent-event-bubbling-in-angular-9-with-setinterval)
- [change-detection-strategy-in-angular](https://dev.to/gaurangdhorda/change-detection-strategy-in-angular-25mc)