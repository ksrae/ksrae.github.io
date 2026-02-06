---
title: "Modern Dependency Injection with Angular Standalone Components"
date: 2024-09-07 19:27:00 +0900
comments: true
categories: angular
tags: [standalone, inject, ondestroy, afterrender, afternextrender]
---

Angular has introduced several features in recent versions that significantly enhance the developer experience. Among these, Standalone Components and the `inject` API are key elements transforming the structure and dependency management of Angular applications. This article explores the impact of these changes on Angular development and introduces new patterns for modern dependency injection using `inject` and component lifecycle management.

## Standalone Components: Using Components Without Module Dependencies

In traditional Angular projects, every component depended on at least one NgModule. However, starting with Angular 15, the concept of Standalone Components was introduced, reducing module dependencies. This allows developers to create more concise and independent components.

### Key Options for Standalone Components

When configuring a Standalone Component, various options are available:

- `standalone: true`: Defines the component as independent without a module.
- `selector`: Defines the tag name to use the component in HTML.
- `templateUrl` and `template`: Sets the HTML template from an external file or as a string.
- `styleUrl`: Previously, style files had to be defined in the `styleUrls` array, but now a single setting is possible.
- `imports`: Imports other Angular modules, directives, and pipes.
- `providers`: Defines services or dependency injections to use in the component.
- `schema`: A setting that determines whether to interact. It is mainly used by the compiler to analyze components and define structural elements or rules that can be allowed in the template. Examples include `NO_ERRORS_SCHEMA` and `CUSTOM_ELEMENTS_SCHEMA`.

```tsx
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  standalone: true,
  selector: 'app-example',
  templateUrl: './example.component.html',
  styleUrl: './example.component.css',
  imports: [CommonModule],
  providers: [HttpService]
})
export class ExampleComponent {}
```

## Evolution of Dependency Injection: The `inject` API

In traditional Angular, dependency injection was achieved through **Constructor Injection**. By injecting dependencies into the constructor of a component or service, Angular's Dependency Injection (DI) system automatically managed objects. For example, the following code was common:

```tsx
import { Component } from '@angular/core';
import { SomeService } from './some.service';

@Component({
  selector: 'app-example',
  templateUrl: './example.component.html'
})
export class ExampleComponent {
  constructor(private someService: SomeService) {}
}
```

While this pattern remains useful, Angular 15 introduced the `inject` API, providing a more flexible way to inject dependencies.

### Features and Usage Examples of the `inject` API

Using `inject` allows you to inject dependencies inside methods or functions rather than just in the class constructor. This enables more flexible code and becomes a powerful tool, especially for asynchronous operations and dynamic service management.

```tsx
import { Component, inject } from '@angular/core';
import { SomeService } from './some.service';

@Component({
  selector: 'app-example',
  templateUrl: './example.component.html'
})
export class ExampleComponent {
  private someService = inject(SomeService);

  someMethod() {
    this.someService.performAction();
  }
}
```

### Why Use `inject`?

- **Flexibility and Conciseness:** Injecting dependencies within methods simplifies code.
- **Ease of Testing:** The structure where dependencies are injected only in specific methods makes testing and reuse easier.
- **Asynchronous Operation Optimization:** Dependency injection can be managed more flexibly in complex asynchronous operations.
- **Zoneless Change Detection:** Supports integration with Angular's new Zoneless change detection method, further optimizing performance.

## Evolution of Component Lifecycle Management: `DestroyRef` for Better Resource Release

In Angular, the `ngOnDestroy` method is typically used to unsubscribe or clean up resources when a component is destroyed. However, Angular 15 introduces a simpler approach with **DestroyRef**, making lifecycle management more intuitive.

### Using `DestroyRef`

`DestroyRef` is provided through dependency injection, and when used with RxJS's `takeUntil` operator, it can automatically unsubscribe when the component is destroyed.

```tsx
import { Component, inject, OnInit } from '@angular/core';
import { DestroyRef } from '@angular/core';
import { interval } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

@Component({
  selector: 'app-example',
  template: `<h1>DestroyRef Example</h1>`
})
export class ExampleComponent implements OnInit {
  private destroyRef = inject(DestroyRef);
  private numbers$ = interval(1000).pipe(takeUntil(this.destroyRef.onDestroy()));

  ngOnInit() {
    this.numbers$.subscribe(num => console.log('Number:', num));
  }
}
```

As shown above, **DestroyRef** automatically cleans up resources when a component is destroyed, preventing memory leaks and reducing code complexity.

## Post-Rendering Tasks: `afterRender` and `afterNextRender`

Angular provides lifecycle hooks like `ngAfterViewInit` and `ngAfterViewChecked` to handle tasks that occur after rendering. Recently, the **afterRender** and `afterNextRender` methods have been introduced, enabling more granular control.

### `afterRender`: Handling Post-Rendering Tasks

`afterRender` is called immediately after the view is rendered and is useful for applying styles and manipulating DOM elements at a point when the DOM elements are fully prepared.

```tsx
@Component({
  standalone: true,
  selector: 'my-chart-cmp',
  template: `<div #chart>{{ ... }}</div>`,
})
export class MyChartCmp {
  @ViewChild('chart') chartRef: ElementRef;
  chart: MyChart|null;

  constructor() {
    afterNextRender(() => {
      this.chart = new MyChart(this.chartRef.nativeElement);
    }, {phase: AfterRenderPhase.Write});
  }
}
```

### `afterNextRender`: Handling Tasks After the Next Rendering Cycle

`afterNextRender` is called after the next rendering cycle is complete, allowing for the efficient handling of UI update tasks after data changes.

```tsx
@Component({
  standalone: true,
  selector: 'my-cmp',
  template: `<span #content>{{ ... }}</span>`,
})
export class MyComponent {
  resizeObserver: ResizeObserver|null = null;
  @ViewChild('content') contentRef: ElementRef;

  constructor() {
    afterNextRender(() => {
      this.resizeObserver = new ResizeObserver(() => {
        console.log('Content was resized');
      });

      this.resizeObserver.observe(this.contentRef.nativeElement);
    }, {phase: AfterRenderPhase.Write});
  }

  ngOnDestroy() {
    this.resizeObserver?.disconnect();
    this.resizeObserver = null;
  }
}
```

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular-Standalone-Component%EC%99%80-%EC%83%88%EB%A1%9C%EC%9A%B4-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85-%EB%B0%A9%EC%8B%9D)
<br/>