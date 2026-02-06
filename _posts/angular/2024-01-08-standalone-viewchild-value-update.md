---
title: "Resolving Delayed Template Updates in Angular"
date: 2024-01-08 15:00:00 +0900
comments: true
categories: angular
tags: [viewchild, template]
---

## Introduction

This article explores how to address the issue of values not immediately reflecting in the template of an Angular application. While typically, declared variables are instantly reflected, certain situations, such as using `OnPush` change detection, can prevent this behavior. This article will not delve into an extensive explanation of change detection but rather directs you to the official Angular documentation: [Angular Change Detection and Optimization](https://angular.io/guide/change-detection).

The issue of delayed template updates often arises when the state updating the variables isn't aligned with the Angular change detection cycle. Several common scenarios contribute to this.

## Utilizing `subscribe`

When dealing with asynchronous operations like RxJS, attempting to subscribe to a value and synchronize it with a variable can result in delayed template updates. Consider the following example:

```tsx
@Component({
  template: `{{ key }}`
})
export class DemoComponent {
  key!: string;

  sampleService = inject(SampleService);
  sample$ = this.sampleService.value$.pipe(
    // ...
  ).subscribe(val => {
      this.key = val; // Update not immediately reflected in the template.
  });
  // ...
}
```

In this scenario, the template might not immediately reflect the updated `key` value after the subscription emits a new value.

## Employing `setTimeout`

Injecting values within a `setTimeout` callback can lead to updates occurring in a microtask, which may not be synchronized with the Angular change detection cycle, thus preventing immediate template reflection.

```tsx
@Component({
  template: `{{ key }}`
})
export class DemoComponent {
  key!: string;

  update(val: any) {
    setTimeout(() => {
      this.key = val; // Update not immediately reflected in the template.
    }, 0); // or any time value
  }
  // ...
}
```

The delay, even with a `0` timeout, introduces an asynchronous operation outside of Angular's regular processing.

## Retrieving Values from `ViewChild`

Attempting to reflect values from a child component obtained using `ViewChild` in the template might not result in immediate updates. This is because this approach deviates from Angular's recommended data handling methods using `@Input` and `@Output`.

```tsx
@Component({
  template: `
    <app-child #child></app-child>
    {{ key }}
  `
})
export class ParentComponent {
  @ViewChild('child') child: AppChildComponent;

  key!: string;

  update() {
    this.key = this.child.value; // Update not immediately reflected in the template.
    this.key = this.child.getValue(); // Getting value from child function not updated yet.
  }
  // ...
}
```

Directly accessing child component properties bypasses Angular's change detection mechanism.

## Outside Angular Zone

There are various situations in which updates might occur outside of the Angular zone, leading to the template not reflecting the changes immediately. Further exploration of these scenarios can be found in dedicated documentation.

## Solutions

To resolve delayed template updates, you can leverage `ChangeDetectorRef.markForCheck()` or `detectChanges()` to control when values are updated. For instance, when using `subscribe`, the following modification can resolve the issue:

```tsx
@Component({
  template: `{{ key }}`
})
export class DemoComponent {
  cdr = inject(ChangeDetectorRef);

  key!: string;

  sampleService = inject(SampleService);
  sample$ = this.sampleService.value$.pipe(
    // ...
  ).subscribe(val => {
      this.key = val;
      this.cdr.markForCheck(); // Immediate update reflected in the template.
  });
  // ...
}
```

`ChangeDetectorRef` provides both `markForCheck` and `detectChanges` functions. Refer to the following resources for understanding the differences between them:

- [Difference between markforcheck and detectchanges](https://stackoverflow.com/questions/41364386/whats-the-difference-between-markforcheck-and-detectchanges)
- [Difference between markforcheck and detectchanges in angular](https://medium.com/@andre.schouten_ff/whats-the-difference-between-markforcheck-and-detectchanges-in-angular-fff4e5f54d34)
- [ChangeDetectorRef](https://angular.io/api/core/ChangeDetectorRef)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular%EC%97%90%EC%84%9C-%EC%A7%80%EC%97%B0%EB%90%9C-%ED%85%9C%ED%94%8C%EB%A6%BF-%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8-%ED%95%B4%EA%B2%B0)
<br/>