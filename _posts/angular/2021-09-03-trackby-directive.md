---
title: "trackBy Directive"
date: 2021-09-03 16:05:00 +0900
comments: true
categories: angular
tags: [ngfor, trackby]
---

When rendering lists in Angular, any change to the data array can cause Angular to re-render the entire DOM for that list. This can lead to performance degradation. To solve this, Angular provides a mechanism to uniquely identify each item, ensuring that only the changed elements are updated.

## The Modern Angular Solution: @for and track
Starting from Angular v17, the new Control Flow syntax, @for, solves this problem very elegantly. It allows you to specify a unique property for tracking each item directly using the track keyword.

```html
<!-- app.component.html -->
<ul>
  @for (item of data(); track item.id) {
    <li>{{ item.name }}</li>
  }
</ul>
```

The track item.id syntax completely eliminates the need for a trackBy function. If you are working on a new project or using a modern version of Angular, using @for with track is always the best approach.


When the value of an array bound to `ngFor` changes, a refresh occurs, triggering re-rendering. To prevent this, we can leverage `trackBy`. Essentially, using `trackBy` can boost performance by preventing re-rendering even when values change.

## Creating a trackBy Directive for the Legacy *ngFor
However, you might still need to use *ngFor in legacy projects or due to compatibility issues with certain libraries. To optimize performance with *ngFor, you must use the trackBy property along with a tracking function (trackByFn).

```html
<!-- app.component.html -->
<div *ngFor="let item of data; trackBy: trackByFn"></div>
```

```tsx
// app.component.ts
...
trackByFn = (index: number, item: any): number => item.id;
```

### Drawbacks of `trackBy`

The main drawback of the conventional `trackBy` approach is that you need to create a `trackByFn` function within the component for every `ngFor` loop. This inconvenience often leads developers to intentionally or accidentally omit `trackBy`, which can negatively impact product performance.

## Understanding `trackBy` Behavior

To create a `trackBy` Directive, let's first understand how `trackBy` works under the hood. We can gain more insight by converting the `*ngFor` syntax to its `ng-template` equivalent. The following two code snippets produce the same result:

```html
{% raw %}
<div *ngFor="let item of data; trackBy: trackByFn"></div>

<ng-template ngFor let-item [ngForOf]="data" [ngForTrackBy]="trackByFn"></ng-template>
{% endraw %}
```

As you can see from the `ng-template` version, the original name of `trackBy` is `ngForTrackBy`, and it's essentially an input binding of the `ngFor` directive.

## Challenges of Creating a `trackBy` Directive

How can we turn `ngForTrackBy` into a directive? A naive approach might look like this:

```html
{% raw %}
<div *ngFor="let item of data" [trackByFn]="'id'"></div>
{% endraw %}
```

Expanding this into its `ng-template` form reveals the problem:

```html
{% raw %}
<ng-template ngFor let-item [ngForOf]="data">
	<div [ngForTrackByFn]="'id'"></div>
</ng-template>
{% endraw %}
```

Using this approach, the custom directive `ngForTrackByFn` is recognized as a separate directive, not an input binding of the `ngFor` directive. Therefore, it cannot influence the `ngFor` behavior, preventing us from achieving the desired functionality.

So, how can we access the `ngFor` directive?

## Creating an `ngForTrackBy` Directive Using `@Host`

To address this, we introduce the `@Host` decorator. `@Host` is a built-in Angular feature that, similar to `@Self`, allows us to retrieve associated dependency injections (DIs).

The key difference is that `@Self` retrieves DIs associated with the component itself, while `@Host` retrieves DIs associated with the component's template and `viewProviders`.

In our case, we can use `@Host` to access the `ngFor` directive and inject the desired field value into the `ngForTrackBy` function belonging to `ngFor`. We'll specify `NgForOf` in `@Host` to ensure we're accessing the `ngFor` directive. Here's the code:

```tsx
// ng-for-track-by-field.directive.ts
import { NgForOf } from '@angular/common';
import { Directive, Input, inject } from '@angular/core';

@Directive({
  // ngFor와 함께 사용될 때만 활성화되도록 셀렉터 구성
  selector: '[ngFor][ngForTrackByField]',
  standalone: true,
})
export class NgForTrackByFieldDirective<T extends Record<string, any>> {
  @Input('ngForTrackByField')
  public field!: keyof T;

  // host 요소에 있는 NgForOf 인스턴스를 주입받습니다.
  private ngFor = inject(NgForOf<T>, { host: true });

  constructor() {
    // NgForOf 디렉티브의 trackBy 함수를 동적으로 설정합니다.
    this.ngFor.ngForTrackBy = (index: number, item: T) => {
      // field가 지정되었다면 해당 프로퍼티 값을, 아니면 아이템 자체를 반환합니다.
      return this.field ? item[this.field] : item;
    };
  }
}
```

In the code above, `ngForTrackByField` is a directive that is hosted by `ngFor`. When invoked, it injects the desired value into the `ngForTrackBy` function of `ngFor` and executes it.

Now, we can easily implement `trackBy` in the template using `trackByField`:

## Using the Directive
Now, you can implement trackBy in your templates without needing a trackByFn in your component.

```Ts
// app.component.ts
import { Component, signal } from '@angular/core';
import { CommonModule } from '@angular/common'; // For *ngFor
import { NgForTrackByFieldDirective } from './ng-for-track-by-field.directive';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, NgForTrackByFieldDirective], // Import the directive
  template: `
    <h3>*ngFor with Custom trackBy Directive</h3>
    <div *ngFor="let item of data(); ngForTrackByField: 'id'">
      {{ item.name }}
    </div>
  `,
})
export class AppComponent {
  data = signal([ { id: 1, name: 'Item 1' }, { id: 2, name: 'Item 2' } ]);
  // No more trackByFn needed here!
}
```

## References

- [Host](https://angular.io/api/core/Host)
- [Host Decorator in Angular](https://www.tektutorialshub.com/angular/host-decorator-in-angular/)
- [How you can use trackBy with a property name](https://medium.com/@ingobrk/here-is-how-you-can-use-trackby-with-a-property-name-ec3bbba8fa75)