---
title: "How To Make Lazy Loading Faster"
date: 2020-10-27 10:00:00 +0900
comments: true
categories: angular
tags: [lazyloading]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EB%B9%A0%EB%A5%B8-Lazy-Loading-%EB%B0%A9%EB%B2%95)
<br/>

## Analysis

Lazy Loading offers the advantage of reducing initial loading time. However, it can introduce a drawback: when a lazily loaded page is accessed, the characteristic instant page transition typical of Single Page Applications (SPAs) might not occur, resulting in a perceived delay.

Is there a solution that can satisfy both requirements?

Here, we present two approaches: `PreloadAllModules` and `QuicklinkStrategy`.

## PreloadAllModules

`PreloadAllModules` differs from the conventional method of loading all modules upfront. Instead, it loads only the minimal amount of code necessary to make all modules runnable during the initial loading phase, allowing them to remain in a suspended state.

This approach is beneficial, offering fast speeds and significantly reduced network usage compared to pre-loading everything. However, in large-scale projects, it can still lead to substantial network overhead. Furthermore, because the router needs to register routes for all preloaded modules, it can trigger intensive computations on the UI thread, potentially degrading the user experience.

Therefore, this approach is recommended for small-scale projects with fewer routes.

### Usage

You can set the options when you initially declare `RouterModule.forRoot`. `PreloadAllModules` is built into `@angular/router`.

```tsx
import { RouterModule, Routes, PreloadAllModules } from '@angular/router';

@NgModule({
  imports: [
    RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules })
  ],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

## QuicklinkStrategy

`QuicklinkStrategy` isn't part of the core Angular library, so you need to install it via npm.

```
npm i ngx-quicklink --save
```

`QuicklinkStrategy` can be a better alternative in large-scale projects.

The principle of operation is as follows:

> 1.  It uses the IntersectionObserver API to identify `routerLink` elements that are currently visible on the screen.
2.  It uses `requestIdleCallback` to wait until the browser is in an idle state.
3.  It determines whether the user is on a slow connection based on `navigator.connection.effectiveType` or has enabled data saving mode via `navigator.connection.saveData`.
4.  It prepares the modules to be lazy-loaded using Angular's prefetching strategy.
> 

`QuicklinkStrategy` differs from `quicklink` in three main ways:

> 1.  While quicklink prefetches resources via `link[rel="prefetch"]` when possible, falling back to XMLHttpRequest, ngx-quicklink solely relies on XMLHttpRequest because of its mechanism of preloading the current modules of the Angular router. Although using `link[rel="prefetch"]`—like in Guess.js—could be a better alternative, users will likely not perceive the difference in most cases.
2.  ngx-quicklink parses and evaluates the content, rather than simply downloading the associated script bundles. This provides a substantial performance boost when the user navigates to the page.
3.  ngx-quicklink downloads all parent modules in order to pre-load every requested module.
> 

### Usage

You can set the options when you initially declare `RouterModule.forRoot`. You need to import it from `ngx-quicklink`.

```tsx
import { RouterModule, Routes } from '@angular/router';
import { QuicklinkStrategy } from 'ngx-quicklink';

@NgModule({
  imports: [
    RouterModule.forRoot(routes, { preloadingStrategy: QuicklinkStrategy })
  ],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

## Additionally: Usage in Modern Angular (v17+)
In the modern Standalone API environment, routing is configured in app.config.ts instead of an NgModule. The preloading strategy is applied using the provideRouter function and a router feature called withPreloading.

### PreloadingAllModules Usage (Standalone API)

```TypeScript
// src/app/app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withPreloading, PreloadAllModules } from '@angular/router';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    // Pass the PreloadAllModules strategy with the withPreloading feature.
    provideRouter(routes, withPreloading(PreloadAllModules))
  ]
};
```

### QuicklinkStrategy Usage (Standalone API)
The ngx-quicklink library provides a provideQuicklink function for Standalone APIs.

```TypeScript
// src/app/app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withPreloading } from '@angular/router';
// Import the necessary provider and strategy from ngx-quicklink.
import { provideQuicklink, QuicklinkStrategy } from 'ngx-quicklink';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withPreloading(QuicklinkStrategy)),
    
    // Provide the Quicklink feature to the application.
    provideQuicklink()
  ]
};
```

## References

- [quicklink-angular-prefetching-preloading-strategy](https://blog.mgechev.com/2018/12/24/quicklink-angular-prefetching-preloading-strategy/)
- [route-preloading-in-angular](https://web.dev/route-preloading-in-angular/)