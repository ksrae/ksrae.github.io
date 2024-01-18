---
title: "Subject를 Unsubscribe 하기 (How to Unsubscribe Subject)"
date: 2019-07-05 16:30:00 +0900
comments: true
categories: angular
tags: [rxjs, subscription, unsubscribe]
---



Subject를 Subscribe를 할 때 Subscription으로 받아둔 뒤 해당 Subscription을 unsubscribe 하는 방법입니다.



## 예시
아래 코드에서는 onInit 일 때 mountSubject를 subscribe 하고, onDestroy 일 때 unsubscribe 하는 코드입니다. 생각보다 간단하므로 쉽게 적용할 수 있습니다.


```ts
import { OnInit, OnDestroy } from '@angular/core';
import { Subject, Subscription } from 'rxjs';

export class SubscribeClass implements OnInit, OnDestroy {
mountSubject: Subject;
mountSubscription: Subscription;

  constructor() {
    this.mountSubject = new Subject();
  }

  ngOnInit() {
    this.mountSubscription = this.mountSubject.subscribe();
  }

  ngOnDestroy() {
    this.mountSubscription.unsubscribe();
  }  
}

```

## 다중 처리

만일 여러 개의 Subscription이 같은 방식으로 동작한다면, <br>
하나의 Subscription으로 여러 subscribe들을 모아 처리할 수 있습니다.<br>
이 경우 add() 함수를 활용합니다. 

```ts

export class SubscribeClass implements OnInit, OnDestroy {
mountSubject: Subject;
anotherMountSubject: Subject;
mountSubscription: Subscription = new Subscription();

  constructor() {
    this.mountSubject = new Subject();
    this.anotherMountSubject = new Subject();
  }

  ngOnInit() {
    this.mountSubscription.add(
      this.mountSubject.subscribe(),
      this.anotherMountSubject.subscribe()
    );
  }

  ngOnDestroy() {
    this.mountSubscription?.unsubscribe();
  }
}
```

각 Subject/Observable의 subscribe 내의 내용이 길어질 경우 함수로 빼서 처리하면, 깔끔하게 정리할 수 있습니다.

```ts
export class SubscribeClass implements OnInit, OnDestroy {
mountSubject: Subject;
anotherMountSubject: Subject;
mountSubscription: Subscription = new Subscription();

  constructor() {
    this.mountSubject = new Subject();
    this.anotherMountSubject = new Subject();
  }

  ngOnInit() {
    this.mountSubscription.add(
      this.mount(),
      this.anotherMount()
    );
  }

  mount = () => {
    return this.mountSubject.subscribe(() => {
      ...
    });
  }

  anotherMount = () => {
    return this.anotherMountSubject.subscribe(() => {
      ...
    });
  }

  ngOnDestroy() {
    this.mountSubscription?.unsubscribe();
  }
}
```

## takeUntilDestroyed (>= Angular 16)
Angular 16 버전부터 OnDestroy를 대신할 수 있는 DestroyRef 를 지원합니다.<br/>
또한 rxjs에서는 takeUntilDestroyed 함수를 지원하는데 DestroyRef와 함께 사용하면 Subscription을 사용하지 않고도 Component의 OnDestroy cycle에 해당 Observable을 unsubscribe 할 수 있습니다.

```ts
import { inject, DestroyRef } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

export class TestComponent {
  destroyRef = inject(DestroyRef);

  ...

  this.sample$.pipe(
    takeUntilDestroyed(this.destroyRef)
  ).subscribe();
}
```