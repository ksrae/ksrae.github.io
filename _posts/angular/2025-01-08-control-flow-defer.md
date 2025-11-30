---
title: "Control flow - defer"
date: 2025-01-08 10:41:00 +0900
comments: true
categories: angular
tags: [controlflow, defer]
---

Deferred views, implemented via the `@defer` block, reduce an application's initial bundle size by delaying the loading of code that isn't strictly necessary for the page's initial render. This commonly results in faster initial load times and improvements to Core Web Vitals (CWV), particularly Largest Contentful Paint (LCP) and Time to First Byte (TTFB).

### Key Concepts

- **User Experience Optimization:** In modern web design, optimizing the user experience during page load is critical. Implement strategies that prioritize the loading of core elements (e.g., videos, images) while deferring less critical ones (e.g., comment sections).
- **Deferred Loading:** While Angular has offered lazy loading capabilities, these could be complex. `@defer` simplifies this, providing a unified deferred loading approach suitable for both client-side and server-side rendering.

### Advantages

- **Performance Improvement:** Pages render faster initially because critical UI elements load first.
- **Resource Conservation:** Conserve resources by only loading elements when they are actually needed.
- **Simplified Implementation:** Implement deferred loading with a simple block structure, avoiding complex lazy loading configurations.

### Usage

`@defer` is used within templates as follows:

```html
@defer(condition) {
  ...deferred stuff
}
```

- `condition`: The content within the block loads when this condition is met. For example, it could be triggered by a button click.
- `deferred stuff`: Components, directives, and pipes referenced within this block are lazily loaded.

## Managing Different Stages of Deferred Loading

### @placeholder

There are situations where you want to display initial content that will later be replaced by code loaded via the `@defer` block.

To achieve this, wrap the initial content in an `@placeholder` block:

```html
@defer {
  <large-component />
}
@placeholder {
  <initial-content />
}
```

When the deferred content is ready, the placeholder is removed, and the `large-component` is displayed in its place.

#### `@placeholder` Parameters

The `@placeholder` block can have one optional parameter called `minimum`, which is a time duration in seconds or milliseconds.

`minimum` is used to set the minimum amount of time the placeholder block is displayed to the user:

```html
@defer {
  <large-component />
}
@placeholder (minimum 2s) {
  <initial-content />
}
```

The code above renders `<initial-content />` for 2 seconds before `<large-component />` appears. Depending on network speed, the `@defer` block might appear too quickly, displaying the placeholder only momentarily. This can result in a strange flickering effect on the page, making the user feel something is broken.

### @loading

The `@loading` block is used to display some content while the `@defer` block is loading the JavaScript bundle in the background.

```html
@defer {
  <large-component />
}
@loading {
  <loading-spinner />
}
```

Once loading is complete, `<loading-spinner />` is removed from the page, and `<large-component />` is rendered in its place.

#### `@loading` Parameters

Accepts `minimum` and `after`:

- `minimum`: Specifies the minimum time the `@loading` block is displayed to the user.
- `after`: Specifies the time to wait before displaying the `@loading` indicator after the loading process has started.

```html
@defer {
  <large-component />
} @loading (after 1s; minimum 2s) {
  <loading-spinner />
}
```

The loading indicator is only displayed if loading takes more than 1 second; otherwise, it will never show. Once the loading indicator is displayed, it remains visible for a minimum of 2 seconds after loading is complete.

#### Difference between `@placeholder` and `@loading`

Although both blocks may seem similar, they serve different purposes.

`@placeholder` is initially displayed until the content of the `@defer` block is ready to be rendered. This block is displayed even before bundle loading begins. Loading might only be triggered after a user clicks a button. Therefore, you might want to display some content to the user until bundle loading is triggered, which is when `@placeholder` comes in handy.

On the other hand, the `@loading` block is only displayed when the `@defer` block's bundle loading starts and is in progress. It disappears once loading is complete.

### @error

The `@error` block is used to display content when the loading of the `@defer` block fails for some reason.

```html
@defer {
  <large-component />
} @error {
  <error-message />
}
```

## when

`when` specifies that a component should only be rendered when a specific condition is met.

The component is rendered only if the expression following `when` is true. This allows for easy implementation of conditional rendering.

```tsx
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-main',
  template: `
    <button (click)="toggleComments()">Show Comments</button>

    @defer(when showComments) {
      <app-comment-section></app-comment-section>
    }
  `
})
export class MainComponent {
  showComments = signal(false);  // Signal to manage the display state of the comment section

  toggleComments() {
    this.showComments.set(!this.showComments());  // Invert the current state
  }
}
```

## trigger

You can write detailed conditions using built-in functions.

| trigger     | Description                                                                                                                                              |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| idle        | Triggers deferred loading when the browser reaches an idle state.                                                                                       |
| interaction | Triggers deferred loading when a click, focus, touch, or input event occurs.                                                                            |
| immediate   | Triggers deferred loading immediately after the client finishes rendering.                                                                               |
| timer(x)    | Triggers deferred loading after a set amount of time has passed. `x` is the time in milliseconds.                                                        |
| hover       | Triggers deferred loading when the mouse hovers over the specified trigger area.                                                                       |
| viewport    | Triggers the deferred block when the specified content enters the viewport.                                                                            |

### on

To use the built-in triggers, you need to use `on`.

#### idle (default; applied as the default value if the condition of defer is omitted)

```html
@defer (on idle) {
  <large-cmp />
}
```

#### interaction

The `interaction` trigger allows you to load deferred content when a user interacts with a specific element. It exposes the defer on click or keydown events.

By default, the placeholder acts as the interaction element. That is, clicking on the placeholder or entering a key triggers the `@defer` block, loading the deferred content.

```html
@defer (on interaction) {
  <large-cmp />
} @placeholder {
  <div>Placeholder</div> // trigger on click
}
```

You can also specify an element to interact with by using a template reference variable within the same template. For example:

```html
<div #greeting>Hello!</div>
@defer (on interaction(greeting)) {
  <greetings-cmp />
}
```

#### immediate

This trigger immediately triggers the `@defer` block. It doesn't wait for any event to occur.

```html
@defer (on immediate) {
  <large-component />
}
```

This means that it triggers this event without waiting for the browser to become idle.

#### timer(x)

This trigger is fired when the timer period is reached.

The timer value can be set in milliseconds (ms) or seconds (s).

```html
@defer (on timer(5s)) {
  <large-component />
}
```

In the code above, the `@defer` block is triggered after 5 seconds.

#### hover

This trigger is fired when the user hovers over an element on the page. The hover event is mouseenter and focusin. This event is only triggered if the `@placeholder` block contains a single node.

```html
@defer (on hover) {
  <large-component />
} @placeholder {
  <loading-spinner />
}
```

In the example below, the `@defer` block is triggered when the `#title` element is hovered or focused.

```html
<div #title>Title</div>

@defer (on hover(title)) {
  <large-component />
}
```

#### viewport

```html
@defer (on viewport) {
  <large-cmp />
} @placeholder {
  placeholder // trigger if this is visible.
}
```

You can also specify a template reference variable within the same template to monitor the element entering the viewport.

```html
<div #greeting>Hello!</div>
@defer (on viewport(greeting)) {
  <greetings-cmp />
}
```

In the code above, when you scroll down the page and the element named `#greeting` appears on the screen, the `@defer` block is triggered, and `greetings-cmp` is loaded.

## Prefetching @defer

Prefetching is the process of loading resources into memory in advance so that they can be used when needed.

```html
@defer (on timer(5s)) {
  <large-component />
}
```

Functionally, it is the same as:

```html
@defer (on timer(5s); prefetch on idle) {
  <large-component />
}
```

The bundle is preloaded when the browser is idle, which is the default behavior. However, `<large-component />` is only displayed to the user after 5 seconds, even if preloading was completed long ago.

The idle trigger is a great default for preloading, but you can also use other built-in triggers you've learned or define custom triggers. Let's look at some examples of what you can do with preloading options.

### Prefetch in the viewport and display on interaction

Imagine you have a large component at the bottom of the page that you only want to load when the user scrolls down the page. Then, you only want to display it when the user enters something into the input box. You could do the following:

```html
@defer (on interaction; prefetch on viewport) {
  <large-component />
} @placeholder {
  <input />
}
```

All built-in triggers can be used for both preloading and displaying the `@defer` block.

### Custom @defer Trigger using the when keyword

If necessary, you can define custom triggers using the `when` keyword.

```tsx
@Component({
  selector: "app",
  template: `
    <button (click)="onLoad()">Preload Trigger</button>
    <button (click)="onDisplay()">Display Trigger</button>

    @defer(when show; prefetch when load) {
      <large-component />
    }
  `,
})
export class AppComponent {
  load: boolean = false;
  show: boolean = false;

  onLoad() {
    this.load = true;
  }

  onDisplay() {
    this.show = true;
  }
}
```

In this example, preloading and display events are configured using custom triggers.

Clicking the preload button triggers the loading of the `@defer` JavaScript bundle in the background. However, the `@defer` block is not yet displayed because the `show` variable is still false.

Then, clicking the display button displays the `@defer` block after the bundle loading is complete. On the other hand, refresh the page and this time click the display button first. In this case, you will notice that preloading is triggered and the block is displayed immediately.

This means that while you can configure custom preloading triggers, if the display trigger is executed first, preloading is performed immediately, and the preloading condition is ignored. This is reasonable because the `@defer` block cannot be displayed if the bundle is not loaded first.

### Difference between @defer and @if

When you use `@defer`, once the logical condition is triggered and the component is loaded and displayed, it cannot be hidden again. `@if` can be hidden again depending on the condition.

The two can also be combined to express it.

```html
@defer (on interaction; prefetch on viewport) {
  @if (someCondition) {
    <large-component />
  }
  @placeholder {
    <placeholder-component />
  }
}
```