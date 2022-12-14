---
title: "StaticInjectorError"
date: 2019-08-19 17:19:00 +0900
comments: true
categories: angular
tags: [universal]
---


StaticInjectorError(AppServerModule)[InjectionToken ng-toolkit-window] 에러를 해결해보겠습니다.


Angular universal에서 window를 사용하기 위해서 `@ng-toolkit/universal`에서 import 합니다.

```ts
import { WINDOW } from '@ng-toolkit/universal';
...
constructor(
  @Inject(WINDOW) public window: Window
)
```

그런데 빌드하면 다음과 같은 에러가 발생합니다.

```
StaticInjectorError(AppServerModule)[InjectionToken ng-toolkit-window]
```

이를 해결하기 위해서는 `@Inject()` 앞에 `@Optional`을 추가하여야 합니다.

```ts
import { Optional } from '@angular/core';
import { WINDOW } from '@ng-toolkit/universal';
...
constructor(
  @Optional() @Inject(WINDOW) public window: Window
)
```


## 참고 사이트
- [Unable to inject window into component · Issue #515 · maciejtreder/ng-toolkit · GitHub](https://github.com/maciejtreder/ng-toolkit/issues/515)
 