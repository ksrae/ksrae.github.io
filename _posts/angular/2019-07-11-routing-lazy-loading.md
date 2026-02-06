---
title: "Angular 8 New Lazy Loading Routing"
date: 2019-07-11 12:55:00 +0900
comments: true
categories: angular
tags: [routing, lazyloading]
---

Angular's lazy loading routing method has been updated.

This change aims to implement a dynamic import approach with Ivy. While the existing method remains valid, it is anticipated that the new approach will be maintained moving forward.

## Changes

The previous lazy loading method involved manually entering the path and module name as a string, which wasn't the most elegant solution.

The new approach involves importing the module link and calling the module through a promise.

```tsx
// Previous
{ path: 'about', loadChildren: '../about/about.module#AboutModule', data: {preload: true} },

// Updated
{ path: 'about', loadChildren: () => import('../about/about.module').then(m => m.AboutModule), data: {preload: true} },
```

Non-lazy-loaded routing remains unchanged, and if you are not using Ivy, you can use either method.

Note that version 8.1 may produce an Ivy error during the build process even when using the updated method.

```
ERROR in Unable to write a reference to AboutComponent in c:/about.component.ts from c:/about.module.ts
```

Unfortunately, Ivy still has some unresolved errors, making it unsuitable for production use at this time.

## Update

The Ivy error mentioned above has been resolved in later versions, allowing it to be used in Ivy.

## References
- [Lazy Loaded Routes Angular V8 with Ivy](https://fireship.io/snippets/lazy-loaded-routes-angular-v8-ivy/)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Lazy-Loading-Routing)
<br/>