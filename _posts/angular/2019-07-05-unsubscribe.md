---
title: " subscribe 중인 Subject를 unsubscribe 하기"
date: 2019-07-05 16:30:00 +0900
comments: true
categories: angular
tags: [rxjs, subscription, unsubscribe]
---



### 명시적 선언
Subject를 Subscribe를 할 때 Subscription으로 받아둔 뒤 해당 Subscription을 unsubscribe 하는 방법이다.


### 예시
아래 코드에서는 onInit 일 때 mountSubject를 subscribe 하고, onDestroy 일 때 unsubscribe 하는 코드이다.


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