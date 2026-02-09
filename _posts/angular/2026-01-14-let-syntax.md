---
title: Introducing let syntax in Angular
date: 2026-01-14 12:00:00 +0900
comments: true
categories: angular
tags: [let]
---

Angular 18.1 introduces the `@let` syntax, which provides a formal way to declare and reuse variables directly within templates. This feature addresses the long-standing friction of using workarounds for local variable assignment.

### 1. The Previous Approach (Before)

To store expression results in a template, developers often relied on the `as` syntax within `*ngIf` or `*ngFor`.

```html
{% raw %} 
<div *ngIf="user$ | async as user">
  {{ user.name }}
</div>
{% endraw %}

```

This required unnecessary DOM elements and could cause issues if the value was `falsy`, preventing the content from rendering at all.

### 2. The @let Syntax (After)

With the new `@let` syntax, you can declare variables independently of structural directives.

```tsx
import { Component } from '@angular/core';
import { AsyncPipe } from '@angular/common';
import { of } from 'rxjs';

@Component({
  selector: 'app-let-example',
  standalone: true,
  imports: [AsyncPipe],
  template: `
    @let user = user$ | async;
    @let isAdmin = user?.role === 'admin';

    @if (user) {
      <section>
        <h1>Welcome, {% raw %} {{ user.name }} {% endraw %}</h1>
        <p>Role: {% raw %} {{ isAdmin ? 'Administrator' : 'User' }} {% endraw %}</p>
        
        @let statusText = user.active ? 'Online' : 'Offline';
        <span>Status: {% raw %} {{ statusText }} {% endraw %}</span>
      </section>
    }
  `
})
export class LetExampleComponent {
  user$ = of({ name: 'Adam Kim', role: 'admin', active: true });
}
```

### 3. Key Takeaways

- **Reactive**: While variables are read-only, they are automatically recomputed during change detection (e.g., when an async pipe emits).
- **Scoped**: Access is restricted to the current view and its children; it does not hoist out of blocks.
- **Control Flow Ready**: It works seamlessly with the new `@if` and `@for` syntax.


## Refernece
[introducing-let-in-angular](https://blog.angular.dev/introducing-let-in-angular-686f9f383f0f)
[variables](https://angular.dev/guide/templates/variables)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular-%ED%85%9C%ED%94%8C%EB%A6%BF%EC%9D%98-%EC%83%88%EB%A1%9C%EC%9A%B4-%EB%B3%80%EC%88%98-%EC%84%A0%EC%96%B8-%EB%B0%A9%EC%8B%9D-let-syntax)
<br/>