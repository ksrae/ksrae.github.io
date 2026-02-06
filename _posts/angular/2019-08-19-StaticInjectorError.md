---
title: "StaticInjectorError"
date: 2019-08-19 17:19:00 +0900
comments: true
categories: angular
tags: [universal]
---

Let's investigate the `@ng-toolkit/universal ERROR: Cannot read property 'unshift' of undefined` error.

To utilize the `window` object within an Angular Universal application, the `@ng-toolkit/universal` package is typically installed.

```
ng add @ng-toolkit/universal
```

However, when attempting installation with Angular 8, an `unshift` error may occur.

```
@ng-toolkit/universal ERROR: Cannot read property 'unshift' of undefined
```

Investigation suggests that this issue stems from a version incompatibility.

The `ng add` command might install version `1.1.21` of `@ng-toolkit/universal`. This version is outdated. Upgrading to the latest version, `7.1.2`, resolves the aforementioned error.

## Reference
- [ERROR: Cannot read property 'unshift' of undefined i'm getting this error when adding SSR to exist angular 6 project · Issue #578 · maciejtreder/ng-toolkit · GitHub](https://github.com/maciejtreder/ng-toolkit/issues/578)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/StaticInjectorErrorAppServerModuleInjectionToken-ng-toolkit-window-%EC%97%90%EB%9F%AC)
<br/>