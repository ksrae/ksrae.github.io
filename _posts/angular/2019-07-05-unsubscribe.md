---
title: "Subject를 Unsubscribe 하기 (How to Unsubscribe Subject)"
date: 2019-07-05 16:30:00 +0900
comments: true
categories: angular
tags: [rxjs, subscription, unsubscribe]
---


Implementing unsubscribe when subscribing to a Subject involves storing the Subscription and then unsubscribing from it when appropriate, such as during component destruction.

## Example

The code below demonstrates how to subscribe to `mountSubject` in `onInit` and unsubscribe in `onDestroy`. The process is relatively straightforward.

```tsx
import { OnInit, OnDestroy } from '@angular/core';
import { Subject, Subscription } from 'rxjs';

export class SubscribeClass implements OnInit, OnDestroy {
  mountSubject: Subject<any>;
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

## Handling Multiple Subscriptions

If multiple Subscriptions behave similarly, they can be managed under a single Subscription. This is achieved using the `add()` function.

```tsx
export class SubscribeClass implements OnInit, OnDestroy {
  mountSubject: Subject<any>;
  anotherMountSubject: Subject<any>;
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

When the logic within each Subject/Observable's subscribe method becomes extensive, extracting it into separate functions can improve code readability and maintainability.

```tsx
export class SubscribeClass implements OnInit, OnDestroy {
  mountSubject: Subject<any>;
  anotherMountSubject: Subject<any>;
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
      // ... your logic here
    });
  }

  anotherMount = () => {
    return this.anotherMountSubject.subscribe(() => {
      // ... your logic here
    });
  }

  ngOnDestroy() {
    this.mountSubscription?.unsubscribe();
  }
}
```

## Utilizing `takeUntilDestroyed` (Angular 16+)

Angular 16 introduces `DestroyRef` as a replacement for `OnDestroy`.  RxJS also provides the `takeUntilDestroyed` function. When used together, they allow unsubscribing from Observables during a Component's destroy cycle without explicitly managing Subscriptions.

```tsx
import { inject, DestroyRef } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

export class TestComponent {
  destroyRef = inject(DestroyRef);

  // ...

  ngOnInit() {
    this.sample$.pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe();
  }
}
```