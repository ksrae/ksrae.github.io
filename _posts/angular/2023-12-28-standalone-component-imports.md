---
title: "Managing Imports in Standalone Angular Components"
date: 2023-12-28 10:20:00 +0900
comments: true
categories: angular
tags: [standalone, update]
---

## Introduction

Proper management of imports for Components, Pipes, and Directives within Angular Standalone Components is essential for maintaining a clean and optimized codebase. This post will explore how to handle Components, Pipes, and Directives in templates, optimize the use of `CommonModule`, manage dynamic Components using `createComponent`, and integrate non-standalone Components within standalone architectures.

## 1. Components, Pipes, and Directives in Templates

Within a Standalone Component's template, every Component, Pipe, and Directive used must be explicitly added to the `imports` array. This ensures that Angular can correctly recognize and render the referenced entities. For Pipes and Directives belonging to `CommonModule`, refer to section 2 for optimization strategies. The following example illustrates importing dependencies in a standard scenario (where Pipes and Directives are *not* part of `CommonModule`):

```tsx
// app.component.ts

@Component({
  template: `
    <div [ngClass]="value | customPipe" customDirective>
      <app-child></app-child>
    </div>
  `,
  standalone: true,
  imports: [
    CustomPipe,
    CustomDirective,
    AppChildComponent
  ]
})
export class AppComponent {}
```

## 2. Optimizing `CommonModule` Usage

When using Pipes or Directives from `CommonModule` in your templates, it's generally more efficient to import specific items individually instead of importing the entire `CommonModule`. For instance, with frequently used features like `ngIf`, `ngFor`, or the `async` pipe, directly import these specific functionalities rather than the complete module. To optimize performance, avoid importing `CommonModule` directly; instead, import only the precisely required items. Below is a list of Directives and Pipes included in `CommonModule`:

**Directive**

- `NgClass`
- `NgComponentOutlet`
- `NgFor`
- `NgForOf`
- `NgIf`
- `NgPlural`
- `NgPluralCase`
- `NgStyle`
- `NgSwitch`
- `NgSwitchCase`
- `NgSwitchDefault`
- `NgTemplateOutlet`

**Pipe**

- `AsyncPipe`
- `CurrencyPipe`
- `DatePipe`
- `DecimalPipe`
- `JsonPipe`
- `KeyValuePipe`
- `LowerCasePipe`
- `UpperCasePipe`
- `TitleCasePipe`
- `PercentPipe`
- `SlicePipe`

## 3. Managing Components Created with `createComponent`

Components dynamically created using `createComponent` do not need to be added to the `imports` array. The Angular framework automatically manages instances of Components created in this manner.

```tsx
// app.component.ts

@Component({
  template: ``,
  standalone: true,
  imports: []
})
export class AppComponent {
  viewContainer = inject(ViewContainerRef);
  componentRef = this.viewContainer.createComponent(AddComponent);
}
```

## 4. Integrating Non-standalone Components

Handling non-standalone Components within a standalone Component requires a careful approach. Since you cannot directly add a non-standalone Component to the `imports` of a standalone Component, follow these steps:

- Create an NgModule.
- Add the non-standalone Component to the `declarations` array of the NgModule.
- Add the non-standalone Component to the `exports` array of the NgModule.
- Import the NgModule into the `imports` array of the standalone Component.

### Example

To illustrate this process further, consider the following example:

**Non-Standalone Component**

```tsx
// shared.component.ts

@Component({
  selector: 'shared-component',
  template: `<p>SHARED COMPONENT</p>`,
})
export class SharedComponent {}
```

**NgModule**

```tsx
// shared.module.ts

@NgModule({
  declarations: [
    SharedComponent
  ],
  exports: [
    SharedComponent
  ]
})
export class SharedModule {}
```

**Standalone Component**

```tsx
// app.component.ts

@Component({
  selector: 'app-root',
  template: `<shared-component></shared-component>`,
  standalone: true,
  imports: [
    SharedModule
  ]
})
export class AppComponent
```

## Conclusion

Managing imports effectively within standalone Components is crucial for maintaining a clean, optimized, and modular codebase. Applying these practices diligently ensures robust code and efficient performance.

## Reference

[Common Module](https://angular.io/api/common/CommonModule)