---
title: "Dynamic Component with ngComponentOutlet"
date: 2021-11-21 23:31:00 +0900
comments: true
categories: angular
tags: [dynamic, component, ngcomponentoutlet]
---

## Displaying Components Conditionally with @if
Traditionally, dynamically swapping components in a template based on a condition can be tricky. A common approach was to list multiple components in the template and use conditional rendering directives like @if or @switch to display only the desired one.

```html
@switch (type()) {
  @case (TYPEA) { <component-a /> }
  @case (TYPEB) { <component-b /> }
  @case (TYPEC) { <component-c /> }
  @case (TYPED) { <component-d /> }
}
```

While this approach is intuitive and readable in the template, it can lead to cluttered template code, especially as the number of conditions grows.

## Dynamic Component Rendering with ngComponentOutlet
ngComponentOutlet is a powerful directive that allows you to dynamically select and render a component in your template from within your component's class (the TS file).
Using it is straightforward: declare *ngComponentOutlet on an element where the component should be rendered and bind it to a component class.

```ts
// app.component.ts
import { Component, signal, Type } from '@angular/core';
import { CommonModule } from '@angular/common'; // For NgComponentOutlet

@Component({ selector: 'a-component', standalone: true, template: `<p>Hello World from A</p>` })
export class AComponent {}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, AComponent],
  template: `
    <ng-container *ngComponentOutlet="component()"></ng-container>
  `
})
export class AppComponent {
  component = signal<Type<any>>(AComponent);
}
```

In this code, AppComponent's component Signal holds a reference to AComponent, so AComponent will be rendered inside the <ng-container>.

## Choosing Among Multiple Components
With ngComponentOutlet, you can easily switch the rendered component based on user interactions, like a button click.

```ts
// app.component.ts
import { Component, signal, Type } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({ selector: 'a-component', standalone: true, template: `<p>Hello World</p>` })
export class AComponent {}
@Component({ selector: 'b-component', standalone: true, template: `<p>Good bye</p>` })
export class BComponent {}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, AComponent, BComponent],
  template: `
    <ng-container *ngComponentOutlet="currentComponent()"></ng-container>
    <button (click)="changeComponent()">Change Component</button>
  `
})
export class AppComponent {
  isA = signal(true);
  currentComponent = signal<Type<any>>(AComponent);

  changeComponent() {
    this.isA.update(val => !val);
    this.currentComponent.set(this.isA() ? AComponent : BComponent);
  }
}
```

## How to Inject Data
One of the most powerful features of ngComponentOutlet is its ability to inject data into the dynamically rendered component, regardless of its type. You can inject data via an Injector and project DOM nodes via the content property.

### Using the Injector
The injector allows you to provide values to the dynamically created child component without using @Input. Using an InjectionToken is a common pattern.

```ts
// title.token.ts
import { InjectionToken } from '@angular/core';
export const TITLE = new InjectionToken<string>('app.title');

// child.component.ts
import { Component, inject } from '@angular/core';
import { TITLE } from './title.token';

@Component({
  standalone: true,
  template: `Complete: {{ titleInjected }}`
})
export class ChildComponent {
  titleInjected = inject(TITLE);
}

// app.component.ts
import { Component, Injector, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ChildComponent } from './child.component';
import { TITLE } from './title.token';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule],
  template: `
    <ng-container *ngComponentOutlet="ChildComponent; injector: myInjector"></ng-container>
  `
})
export class AppComponent {
  ChildComponent = ChildComponent; // Expose to template
  myInjector: Injector;

  constructor() {
    // Create a new injector that inherits from the parent injector
    this.myInjector = Injector.create({
      providers: [{ provide: TITLE, useValue: 'hello world from injector' }],
      parent: inject(Injector) // Set the current injector as the parent
    });
  }
}
```

This method is highly flexible, as it allows you to inject various provider types like useClass and useFactory.

### Using content for Projection
The content option lets you project DOM nodes into the <ng-content> slots of the dynamic component. You can pass nodes created with document.createElement or document.createTextNode.<br/>

Note: The value passed to content must be a 2D array (Node[][]).

```ts
// child.component.ts (with ng-content)
@Component({
  standalone: true,
  template: `Complete: <ng-content></ng-content>`
})
export class ChildComponent {}

// app.component.ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule],
  template: `
    <ng-container *ngComponentOutlet="ChildComponent; content: myContent"></ng-container>
  `
})
export class AppComponent {
  ChildComponent = ChildComponent;
  myContent: Node[][];

  constructor() {
    const button = document.createElement('button');
    button.textContent = 'Click me from parent';
    button.onclick = () => alert('Button clicked!');
    
    // Pass the node as a 2D array
    this.myContent = [[button]];
  }
}
```

This allows you to pass DOM elements created and controlled by the parent component to the dynamic child.

## Removing a Rendered Component
You can remove a rendered component by simply assigning null to the component variable bound to ngComponentOutlet.

```ts
// app.component.ts
// ... imports
export class AppComponent {
  // Use a union type of Type<any> | null
  currentComponent = signal<Type<any> | null>(AComponent);

  removeComponent() {
    this.currentComponent.set(null);
  }
}
```

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/ngComponentOutlet%EC%9C%BC%EB%A1%9C-%EB%8F%99%EC%A0%81-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0)
<br/>