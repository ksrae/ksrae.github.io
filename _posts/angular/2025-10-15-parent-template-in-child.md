---
title: Parent template in child with exportAs
date: 2025-10-15 12:00:00 +0900
comments: true
categories: angular
tags: [exportas]
---

[한국어(Korean) Page](https://velog.io/@ksrae/exportAs%EB%A1%9C-%EC%9E%90%EC%8B%9D-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EC%9D%98-%EA%B2%B0%EA%B3%BC%EB%AC%BC%EC%9D%84-%EB%B6%80%EB%AA%A8-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EC%97%90%EC%84%9C-%EB%9E%9C%EB%8D%94%EB%A7%81-%ED%95%98%EA%B8%B0)
<br/>

When designing components in Angular, we often face the challenge of separating responsibilities between parent and child components. This is especially true when "the data and logic belong to the child, but the resulting UI needs to be rendered in a specific location within the parent."<br/>

In this article, we'll explore how to use a lesser-known feature, exportAs, to render a template defined by a child component within its parent in a clean, declarative way.

## The Problem: Whose Responsibility Is It?

Let's consider two common scenarios:

1. **A Window Component with Dynamic Menus:** You have a `<app-window>` component that can contain various child components. The menu items at the top of the window should be determined by the type or state of the child component inside it.
2. **A List with Item-Specific Action Buttons:** An `<app-list>` component renders several `<app-list-item>` s. Each item, based on its own state (e.g., 'editing', 'pending approval'), should have different action buttons like 'Save', 'Delete', or 'Approve', and these buttons need to appear in a common area of the list, like a header or footer.

In both cases, the component that best knows *what* to render is the **child**, but the location where it needs to be rendered belongs to the **parent's** DOM structure. How can we connect these two naturally?

## The Solution: Exposing the Component Instance with exportAs

The key to solving this lies in the exportAs property of the @Component decorator. This property assigns a "nickname" to the component's instance, allowing it to be referenced from within a template.

### 1. Setting Up the Child Component

First, the child component defines the template it wants to provide to the parent using `<ng-template>` and exposes it as a public property on its class via @ViewChild.

```jsx
// child.component.ts
import { Component, ViewChild, TemplateRef } from '@angular/core';

@Component({
  selector: 'app-child',
  standalone: true,
  exportAs: 'smartChild', // 👈 Allows the instance to be referenced as 'smartChild' in a template
  template: `
    <div>Child's unique content goes here.</div>

    <!-- The template we want to pass to the parent -->
    <ng-template #menuItems>
      <button>Item 1</button>
      <button>Item 2</button>
    </ng-template>
  `,
})
export class ChildComponent {
  // Get the template (#menuItems) as a public property (menuItems)
  @ViewChild('menuItems', { static: true }) 
  menuItems!: TemplateRef<any>;
}
```

**Key Points:**

- exportAs: 'smartChild': This allows any parent template using this component to get a reference to this component's instance using the syntax `#variable="smartChild"` .
- `@ViewChild('menuItems') menuItems` : This assigns the template reference #menuItems from the template to the menuItems class property. Since this property is public, it's accessible from the outside (the parent).

### 2. Rendering the Template in the Parent Component

Now, the parent component can get the child's instance as a variable and access its public properties to render the template wherever it desires.

```jsx
// parent.component.ts
import { Component } from '@angular/core';
import { NgTemplateOutlet } from '@angular/common';
import { ChildComponent } from './child.component';

@Component({
  selector: 'app-parent',
  standalone: true,
  imports: [ChildComponent, NgTemplateOutlet],
  template: `
    <div class="parent-layout">
      <header class="parent-header">
        <h2>Parent Component Area</h2>
        <div class="menu-bar">
          <!-- 💡 The location where the child's template will be rendered -->
          <ng-container [ngTemplateOutlet]="child.menuItems"></ng-container>
        </div>
      </header>

      <main>
        <!-- Render the child component and assign its instance to the #child variable -->
        <app-child #child="smartChild"></app-child>
      </main>
    </div>
  `,
})
export class ParentComponent {}
```

**How It Works, Step-by-Step:**

1. `<app-child #child="smartChild"></app-child>`: This creates the `<app-child>` component. Crucially, by using the nickname defined in exportAs (smartChild), it **captures the entire instance of ChildComponent into a template variable named child**.
2. `<ng-container [ngTemplateOutlet]="child.menuItems"></ng-container>`: Elsewhere in the parent's template, we use the ngTemplateOutlet directive.
3. `[ngTemplateOutlet]="child.menuItems"`: We tell ngTemplateOutlet to render the template provided by the menuItems property of our child variable (which is the child component instance). This menuItems property is the exact `<ng-template>` we captured with @ViewChild in the child component.

## Conclusion

exportAs is a powerful technique that allows for flexible control over template rendering between parent and child components without increasing their coupling.<br/>

Instead of using @Output to emit a TemplateRef or relying on a complex shared service, you can define the entire relationship declaratively within the template, making the code more intuitive and clean. This approach enables you to design more flexible and reusable component architectures.

## Reference
[**Angular: How to Render a Template for the Parent in a Child Component**](https://medium.com/@eugeniyoz/angular-how-to-render-a-template-for-the-parent-in-a-child-component-7cf7178a6f17)