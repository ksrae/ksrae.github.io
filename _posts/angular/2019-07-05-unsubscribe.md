---
title: " How to Unsubscribe Subject"
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