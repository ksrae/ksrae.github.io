---
title: "Create Tooltip with createComponent"
date: 2022-02-10 17:46:00 +0900
comments: true
categories: angular
tags: [dynamic, component, createcomponent, tooltip]
---

Let's build a dynamic tooltip that appears on hover, leveraging Angular's powerful createComponent API for dynamic component rendering.

# Basic Operation
- Create two tooltip components (ATooltipComponent, BTooltipComponent) as standalone.
- Create a container component to host the elements that will trigger the tooltips, also as standalone.
- Create a standalone directive that listens for mouseenter and mouseleave events, then dynamically creates and destroys the appropriate tooltip component using createComponent.

# Creating the Components
## The Container Component
We'll create a host for our tooltips and apply our custom tooltip directive to trigger them.

```ts
// src/app/container.component.ts
import { Component } from '@angular/core';
import { TooltipDirective } from './tooltip.directive';

@Component({
  selector: 'app-container',
  standalone: true,
  imports: [TooltipDirective], // Import the directive to use it in the template
  template: `
    <h2>Hover over these elements</h2>
    <div class="box" [tooltip]="'A'">Show Tooltip A</div>
    <div class="box" [tooltip]="'B'">Show Tooltip B</div>
  `,
  styles: `
    .box {
      border: 1px solid #ccc;
      padding: 20px;
      margin: 20px;
      cursor: pointer;
      width: 200px;
      text-align: center;
    }
  `
})
export class ContainerComponent {}
```

## The Dynamic Tooltip Components
These are the simple components that will be dynamically rendered as tooltips.

```ts
// src/app/a-tooltip.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'a-tooltip',
  standalone: true,
  template: `<div class="tooltip">Tooltip A: Hello World</div>`,
  styles: `.tooltip { background-color: lightblue; padding: 10px; border-radius: 5px; }`
})
export class ATooltipComponent {}
```


```ts
// src/app/b-tooltip.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'b-tooltip',
  standalone: true,
  template: `<div class="tooltip">Tooltip B: Goodbye</div>`,
  styles: `.tooltip { background-color: lightcoral; color: white; padding: 10px; border-radius: 5px; }`
})
export class BTooltipComponent {}
```

## Creating the Directive
This is the core logic of our example. The directive detects events and, based on its input ('A' or 'B'), dynamically creates and destroys the correct tooltip component.

```ts
// src/app/tooltip.directive.ts
import {
  Directive,
  Input,
  HostListener,
  ComponentRef,
  ViewContainerRef,
  inject,
  Type
} from '@angular/core';
import { ATooltipComponent } from './a-tooltip.component';
import { BTooltipComponent } from './b-tooltip.component';

@Directive({
  selector: '[tooltip]',
  standalone: true,
})
export class TooltipDirective {
  @Input('tooltip') type: 'A' | 'B' | string = '';

  private componentRef?: ComponentRef<any>;
  // Inject ViewContainerRef using the inject function
  private viewContainerRef = inject(ViewContainerRef);

  @HostListener('mouseenter')
  onMouseEnter(): void {
    if (this.componentRef) return; // Don't create if a tooltip already exists

    const componentToCreate = this.getComponentType(this.type);
    if (!componentToCreate) return;

    // Pass the component class directly, no ComponentFactoryResolver needed
    this.componentRef = this.viewContainerRef.createComponent(componentToCreate);
  }

  @HostListener('mouseleave')
  onMouseLeave(): void {
    if (this.componentRef) {
      this.componentRef.destroy();
      this.componentRef = undefined;
    }
  }

  private getComponentType(type: string): Type<any> | null {
    switch (type) {
      case 'A':
        return ATooltipComponent;
      case 'B':
        return BTooltipComponent;
      default:
        return null;
    }
  }
}
```

- inject(ViewContainerRef): This is the modern, constructor-less way to handle dependency injection.
- createComponent(componentToCreate): Since Angular v13, you can pass the component class directly to create an instance, which makes the code much cleaner and removes the need for ComponentFactoryResolver.
- componentRef.destroy(): This safely removes the created component instance from the DOM and helps prevent memory leaks.

# Note: Configuration in a Standalone Environment
Unlike the old NgModule approach, in a standalone world, each component and directive must explicitly declare its dependencies in its imports array.

- The ContainerComponent uses the TooltipDirective in its template, so it must add TooltipDirective to its imports array.
- The TooltipDirective programmatically creates ATooltipComponent and BTooltipComponent. Since it doesn't use them in a template, it doesn't need to add them to an imports array. The standard TypeScript import statement is sufficient to get their class types.

Now, when you run this example and hover over each div, you'll see the corresponding tooltip dynamically created and destroyed.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/createComponent%EB%A1%9C-%ED%88%B4%ED%8C%81-%EB%A7%8C%EB%93%A4%EA%B8%B0)
<br/>