---
title: "Dynamic Component with createComponent"
date: 2022-02-07 17:46:00 +0900
comments: true
categories: angular
tags: [dynamic, component, createcomponent]
---

Implementing dynamic components with `createComponent` offers a flexible approach to component management, especially useful in more versatile versions.

# Core Principles

The following outlines the fundamental principles involved in dynamically creating components:

1. **Component Factory Retrieval:** Utilize the `componentFactory` to obtain the necessary information about the component you intend to instantiate.
2. **Container's View Container Reference:** Access the `viewContainerRef` of the container component where the dynamic component will reside.
3. **Component Rendering:** Employ the `createComponent` function of the `viewContainerRef` to render the target component within the container.
4. **Component Removal:** Use the `clear` function of the `viewContainerRef` to remove the dynamically created component when it's no longer needed.

# Creating Components

## Container Component

Let's craft a container component responsible for hosting the dynamic components. The example below showcases rendering different components upon clicking respective buttons.

```tsx
@Component({
  selector: 'app-container',
  template: `
  <button (click)="callAComponent()">Show A-Component</button>
  <button (click)="callBComponent()">Show B-Component</button>
  `
})
export class ContainerComponent {
  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver
  ) {}

  callAComponent() {
    const resolve = this.resolver.resolveComponentFactory(AComponent);
    this.viewContainerRef.createComponent(resolve);
  }
  callBComponent() {
    const resolve = this.resolver.resolveComponentFactory(BComponent);
    this.viewContainerRef.resolveComponentFactory(BComponent);
    this.viewContainerRef.createComponent(resolve);
  }
}
```

## Dynamic Components

Now, let's define the two dynamic components we'll be rendering. While CSS styling could enhance visual clarity, it's omitted here for brevity.

```tsx
@Component({
  selector: 'a-component',
  template: `
    <p>Hello World</p>`
})
export class AComponent {

}

@Component({
  selector: 'b-component',
  template: `
    <p>Good bye</p>`
})
export class BComponent {

}
```

Executing this example allows you to toggle between `AComponent` and `BComponent` by clicking the respective buttons.

## Injecting Values

Injecting values into dynamic components is straightforward. You can access the instance returned by `createComponent` and set the desired values on its properties.

In essence, the "instance" refers to the public variables or functions of the component you're instantiating.

Furthermore, you can inject values into variables decorated with `@Input()`. However, note that this approach does *not* work with `@Input('') set ...` and doesn't trigger the `onchange` lifecycle hook. Exercise caution when using this method.

Let's rewrite the components to demonstrate value injection.

### Container Component

```tsx
@Component({
  selector: 'app-container',
  template: `
  <button (click)="callAComponent()">Show A-Component</button>
  <button (click)="callBComponent()">Show B-Component</button>
  `
})
export class ContainerComponent {
  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver
  ) {}

  callAComponent() {
    const resolve = this.resolver.resolveComponentFactory(AComponent);
    const componentRef = this.viewContainerRef.createComponent(resolve);
    componentRef.instance.data = 'hello world';
  }
  callBComponent() {
    const resolve = this.resolver.resolveComponentFactory(BComponent);
    const componentRef = this.viewContainerRef.createComponent(resolve);
    componentRef.instance.data = 'good bye';
  }
}
```

### Dynamic Components

```tsx
@Component({
  selector: 'a-component',
  template: `
    <p>{{data}}</p>`
})
export class AComponent {
  data!: string;
}

@Component({
  selector: 'b-component',
  template: `
    <p>{{data}}</p>`
})
export class BComponent {
  @Input() data!: string;
}
```

Upon execution, you'll observe the injected values displayed via the `data` variable. The `@Input()` in `BComponent` serves to demonstrate compatibility with this injection method.

## Cases Where Value Injection Fails

If you attempt to inject values more than once, subsequent injections via the instance may not reflect changes. While `afterViewInit` might show the value being received, a `changeDetection` cycle is required to render the updated value on the screen.

### Container Component

```tsx
...
export class ContainerComponent implements AfterViewInit {
  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver,
    private cd: ChangeDetectionRef
  ) {}
  ngAfterViewInit() {
    this.cd.markForCheck();
  }
```

## Value Injection using Subjects

If you're hesitant to use `changeDetection`, employing a Subject can be a viable alternative. Utilizing `BehaviorSubject`, which can hold an initial value, simplifies value injection.

### Container Component

```tsx
@Component({
  selector: 'app-container',
  template: `
  <button (click)="callAComponent()">Show A-Component</button>
  <button (click)="callBComponent()">Show B-Component</button>
  `
})
export class ContainerComponent {
  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver
  ) {}

  callAComponent() {
    const resolve = this.resolver.resolveComponentFactory(AComponent);
    const componentRef = this.viewContainerRef.createComponent(resolve);
    componentRef.instance.data$.next('hello world');
  }
  callBComponent() {
    const resolve = this.resolver.resolveComponentFactory(BComponent);
    const componentRef = this.viewContainerRef.createComponent(resolve);
    componentRef.instance.data.next('good bye');
  }
}
```

### Dynamic Components

```tsx
@Component({
  selector: 'a-component',
  template: `
    <p>{{data$ | async}}</p>`
})
export class AComponent {
  data$ = new BehaviorSubject<any>('');
}

@Component({
  selector: 'b-component',
  template: `
    <p>{{data$ | async}}</p>`
})
export class BComponent {
  data$ = new BehaviorSubject<any>('');
}
```

# Removing Components

Component removal is straightforward: simply call the `clear` function of the container component's `viewContainerRef`.

### Container Component

```tsx
@Component({
  selector: 'app-container',
  template: `
  <button (click)="callAComponent()">Show A-Component</button>
  <button (click)="callBComponent()">Show B-Component</button>
  <button (click)="removeAll()">RemoveAll</button>
  `
})
export class ContainerComponent {
  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver
  ) {}

  callAComponent() {
    const resolve = this.resolver.resolveComponentFactory(AComponent);
    const componentRef = this.viewContainerRef.createComponent(resolve);
    componentRef.instance.data$.next('hello world');
  }
  callBComponent() {
    const resolve = this.resolver.resolveComponentFactory(BComponent);
    const componentRef = this.viewContainerRef.createComponent(resolve);
    componentRef.instance.data.next('good bye');
  }
  removeAll() {
    this.viewContainerRef.clear();
  }
}
```

# Additional Notes

## Improvements since v13

Since v13, the original `createComponent` function has been deprecated, and a newly written `createComponent` function must be used. Previously, you had to access the component through `componentFactory`, but the new version allows you to directly access the component, enabling more concise coding.

Rewriting the above container component in v13 would look like this:

### Container Component

```tsx
@Component({
  selector: 'app-container',
  template: `
  <button (click)="callAComponent()">Show A-Component</button>
  <button (click)="callBComponent()">Show B-Component</button>
  `
})
export class ContainerComponent {
  constructor(
    private viewContainerRef: ViewContainerRef
  ) {}

  callAComponent() {
    const componentRef = this.viewContainerRef.createComponent(AComponent);
    componentRef.instance.data$.next('hello world');
  }
  callBComponent() {
    const componentRef = this.viewContainerRef.createComponent(BComponent);
    componentRef.instance.data.next('good bye');
  }
}
```

## Improvements with Modern Angular (v17+)
with v17+, Standalone Components and Signals have become the standard. In modern Angular, creating dynamic components is much more concise and powerful.<br/>
Below is an updated example using Standalone Components, Signals, and @ViewChild.

### 1. The Dynamic Components (A & B Components)
- Declare them as standalone: true.
- Use @Input combined with signal to receive data. The signal is a WritableSignal, which can be modified from the outside.

```Ts
// a.component.ts
import { Component, Input, signal, WritableSignal } from '@angular/core';

@Component({
  standalone: true,
  selector: 'a-component',
  template: `<p>A Component says: {{ data() }}</p>`
})
export class AComponent {
  @Input() data: WritableSignal<string> = signal('');
}
```


```Ts
// b.component.ts
import { Component, Input, signal, WritableSignal } from '@angular/core';

@Component({
  standalone: true,
  selector: 'b-component',
  template: `<p>B Component says: {{ data() }}</p>`
})
export class BComponent {
  @Input() data: WritableSignal<string> = signal('');
}
```

### 2. The Container Component
- Use an ng-container with a template reference variable (#container) in the template to mark the insertion point.
- Access the ViewContainerRef using @ViewChild.
- Pass the component class directly to createComponent to create an instance.
- Inject data by calling the .set() method on the component instance's data signal.

```Ts
// container.component.ts
import { Component, ViewChild, ViewContainerRef } from '@angular/core';
import { AComponent } from './a.component';
import { BComponent } from './b.component';

@Component({
  selector: 'app-container',
  standalone: true,
  imports: [AComponent, BComponent], // Dynamic components must also be imported
  template: `
    <button (click)="callAComponent()">Show A-Component</button>
    <button (click)="callBComponent()">Show B-Component</button>
    <button (click)="removeAll()">Remove All</button>
    
    <!-- The location where dynamic components will be rendered -->
    <ng-container #container></ng-container>
  `
})
export class ContainerComponent {
  // Read the #container element as a ViewContainerRef
  @ViewChild('container', { read: ViewContainerRef, static: true })
  private viewContainerRef!: ViewContainerRef;

  callAComponent() {
    this.viewContainerRef.clear(); // Clear previous components
    const componentRef = this.viewContainerRef.createComponent(AComponent);
    // Set the value of the signal
    componentRef.instance.data.set('Hello from Container!');
  }

  callBComponent() {
    this.viewContainerRef.clear(); // Clear previous components
    const componentRef = this.viewContainerRef.createComponent(BComponent);
    // Set the value of the signal
    componentRef.instance.data.set('Goodbye from Container!');
  }

  removeAll() {
    this.viewContainerRef.clear();
  }
}
```

As you can see, modern Angular allows you to create dynamic components with type safety without ComponentFactoryResolver, and you can easily inject reactive data using Signals.

# References

- [Angular Official Site dynamic-component-loader](https://angular.io/guide/dynamic-component-loader)