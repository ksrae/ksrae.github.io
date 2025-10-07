---
title: "Rxjs Pipe Operators"
date: 2019-07-05 15:00:00 +0900
comments: true
categories: angular
tags: [rxjs, subscribe]
---


## `take(n)`

**Description:** Emits only the first `n` values emitted by the source Observable, then completes.

[](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile30.uf.tistory.com%2Fimage%2F997D983359BA8B6804950D)

Marble Diagram: take(n)

```tsx
this.mount.pipe(take(2)).subscribe();
```

**Usage Notes:**  Useful for scenarios where you only need a limited number of emissions from an observable.

## `takeUntil(notifier: Observable)`

**Description:** Emits the values from the source Observable until the `notifier` Observable emits a value or completes.  Upon notification, `takeUntil` unsubscribes from the source Observable.

[](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile3.uf.tistory.com%2Fimage%2F99591B3359BB22561F61F7)

Marble Diagram: takeUntil(Observable)

```tsx
unsubscribe: Subject<void> = new Subject<void>();

public ngOnInit() {
  this.tagService.getTags({category: 'keyword'})
    .pipe(takeUntil(this.unsubscribe))
    .subscribe(tags => {
      this.tags = tags;
    });
}

public ngOnDestroy() {
  this.unsubscribe.next();
  this.unsubscribe.complete();
}
```

**Usage Notes:** A common pattern in Angular components to unsubscribe from observables when the component is destroyed, preventing memory leaks.  The `notifier` is often a `Subject` that emits a value in the `ngOnDestroy` lifecycle hook.

## `takeWhile(predicate: function)`

**Description:** Emits values from the source Observable as long as the `predicate` function returns `true`.  Once the `predicate` returns `false`, the Observable completes.

[](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile9.uf.tistory.com%2Fimage%2F99F6553359BA8BA230F5B0)

Marble Diagram: takeWhile(function)

```tsx
alive = true;

ngOnInit() {
  this.filterChange$
    .pipe(takeWhile(() => this.alive))
    .subscribe(filter => {
      // Handle filter changes
    });

  this.keywordChange$
    .pipe(takeWhile(() => this.alive))
    .subscribe(keyword => {
      // Handle keyword changes
    });
}

ngOnDestroy() {
  this.alive = false;
}
```

**Usage Notes:** Similar to `takeUntil`, but uses a synchronous predicate function to determine when to unsubscribe.

## `takeLast(n)`

**Description:** Emits only the last `n` values emitted by the source Observable upon completion. Requires the source Observable to complete to emit any values.

[](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/takeLast.png)

Marble Diagram: takeLast.png

**Usage Notes:**  Useful when you need to process the final `n` values of a finite observable stream.

## `first()`

**Description:** Emits only the first value emitted by the source Observable, then completes.  Optionally accepts a predicate function to filter values.

[](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/first.png)

Marble Diagram: first.png

```tsx
this.mount.pipe(first()).subscribe();
```

**Usage Notes:**  Useful for retrieving the first value from an Observable and then completing the subscription.

## `take(predicate: function)`

**Description:** **Note:** This is functionally equivalent to `first(predicate)`. Emits the first value that satisfies the provided `predicate` function, then completes.

[](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/take.png)

Marble Diagram: take.png

```tsx
this.mount.pipe(take(2)).subscribe(); // This example is misleading; it would be better with a predicate.
```

**Usage Notes:** Although named `take`, when used with a function, it behaves like `first` and emits only one value that satisfies the condition. Be careful to distinguish this usage from `take(n)`.

## `skip(n)`

**Description:** Skips the first `n` values emitted by the source Observable, then emits the remaining values.

[](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/skip.png)

Marble Diagram: skip.png

**Usage Notes:**  Useful when you need to ignore a known number of initial emissions.

## `last(predicate?: function)`

**Description:** Emits only the last value emitted by the source Observable (or the last value that satisfies the optional `predicate` function) upon completion.  Effectively equivalent to `takeLast(1)` when no predicate is used.

[](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/last.png)

Marble Diagram: last.png

**Usage Notes:** Similar to `takeLast(1)`, but can also be used with a predicate to find the last value that matches a specific condition.

## `skipLast(n)`

**Description:** Skips the last `n` values emitted by the source Observable.  Requires the source Observable to complete to emit any values.

[](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/skipLast.png)

Marble Diagram: skipLast.png

**Usage Notes:**  Useful when you need to ignore a known number of final emissions of a finite observable.

## `skipUntil(notifier: Observable)`

**Description:** Skips values emitted by the source Observable until the `notifier` Observable emits a value or completes.  After the `notifier` emits, `skipUntil` starts emitting values from the source Observable.

[](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/skipUntil.png)

Marble Diagram: skipUntil.png

**Usage Notes:**  Useful when you want to start processing values only after a specific event has occurred.

## `skipWhile(predicate: function)`

**Description:** Skips values emitted by the source Observable as long as the `predicate` function returns `true`.  Once the `predicate` returns `false`, `skipWhile` starts emitting all subsequent values from the source Observable.

[](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/skipWhile.png)

Marble Diagram: skipWhile.png

**Usage Notes:** Useful when you want to start processing values only after a specific condition is no longer met.

*The content above references the official RxJS documentation.*

- [RxJS Official Documentation](https://rxjs-dev.firebaseapp.com/api/operators)