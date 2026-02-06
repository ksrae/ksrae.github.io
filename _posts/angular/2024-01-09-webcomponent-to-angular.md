---
title: "How to use webcomponent in Angular"
date: 2024-01-09 01:14:00 +0900
comments: true
categories: angular
tags: [webcomponent]
---

In this article, we'll explore how to use web components within an Angular project. We'll focus particularly on invoking web components from Angular components, rather than directly within `index.html`.

## Web Component Setup

Refer to [this previous article](https://ksrae.github.io/angular/webcomponent/) for guidance on creating your web component.

## Project Initialization

Let's create a new Angular project to integrate the web component.

```
npm new angular-webcomponent
```

## Script File Integration

Copy the three files associated with your web component (typically `bundle.js`, `main.js`, and `polyfills.js`) into the new project. The directory location isn't critical; for this example, we've created a `src/scripts` folder.

(If you installed the web component via npm, you can disregard this step.)

## Web Component Tag Invocation

In any Angular component, add the web component tag.

```html
<my-library-component></my-library-component>
```

## Script Import Strategies

Angular provides three methods for including external scripts:

- Adding to `angular.json`'s `"scripts"` section.
- Using `<script>` tags in `index.html`.
- Importing within the component itself.

### Attempting to Add to `angular.json` (Failure)

While you can add script files to the `scripts` array in `angular.json`:

```json
...
"scripts": [
  "./src/scripts/bundle.js"
]
```

This approach typically results in the following warnings and the script will not be applied correctly:

```
Failed to resolve dependency: ./main.js, present in 'optimizeDeps.include'
Failed to resolve dependency: ./polyfills.js, present in 'optimizeDeps.include'
```

Placing the files in the `assets` folder for external access doesn't resolve this. Moreover, including `main.js` and `polyfills.js` often leads to duplicate function errors, rendering this method impractical.

### Adding to `index.html` (Success, but Suboptimal)

You can include script files using `<script>` tags within `index.html`. Placing the scripts within the `assets` folder is a common practice.

```html
...
<script src="./assets/scripts/bundle.js"></script>
```

However, this method loads the script on every page, which is not ideal for optimization. It's generally recommended only for small projects or frequently used components.

### Importing within the Component (Success, Recommended)

The most efficient approach is to directly import the JavaScript file within the component:

```tsx
import "./src/scripts/bundle.js";
```

This ensures the script is only loaded when the component is used, optimizing performance. Use this strategy, especially in larger projects, only for components where the script is required.

## Configuring `CUSTOM_ELEMENTS_SCHEMA`

After defining the web component and importing it, the web component might still not load correctly. This is because web components aren't standard Angular components. To resolve this, you must configure the schema to allow external components.

There are two schema options: `CUSTOM_ELEMENTS_SCHEMA`, which allows custom elements, and `NO_ERRORS_SCHEMA`, which suppresses errors related to unknown elements and attributes.

While `NO_ERRORS_SCHEMA` may seem convenient, it can mask important error messages. Therefore, carefully consider its implications before use.

```tsx
import { CUSTOM_ELEMENTS_SCHEMA, Component } from '@angular/core';

@Component({
  ...
  schemas: [CUSTOM_ELEMENTS_SCHEMA]
})
...
```

With this configuration, the web component should now load successfully.

Below is an example of a component's source code:

```tsx
import { CUSTOM_ELEMENTS_SCHEMA, Component } from '@angular/core';
import "../scripts/bundle.js";

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [],
  template: `<my-library-component></my-library-component>`,
  styleUrl: './app.component.scss',
  schemas: [CUSTOM_ELEMENTS_SCHEMA]
})
export class AppComponent {
  title = 'webcomponent-angular';
}

```

## Resources

[Using Web Components in an Angular application: Joyful & Fun](https://developer.vonage.com/en/blog/using-web-components-in-an-angular-application-joyful-fun)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular%EC%97%90-%EC%9E%91%EC%84%B1%ED%95%9C-Web-Component-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
<br/>