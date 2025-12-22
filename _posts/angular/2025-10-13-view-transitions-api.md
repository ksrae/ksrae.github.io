---
title: What is View Transitions API?"
date: 2025-10-13 12:00:00 +0900
comments: true
categories: javascript
tags: [transition]
---

[한국어(Korean) Page](https://velog.io/@ksrae/View-Translations-API%EB%9E%80)
<br/>

Angular 17 introduced several groundbreaking features that elevate the web development experience. Among them, one of the most exciting is the native support for the View Transitions API.<br/>

In this article, we'll explore what the View Transitions API is, how to use it in Angular, and how it can significantly enhance the user experience.

## What is the View Transitions API?

The View Transitions API is a web standard API that makes it easy to implement smooth, animated transitions between different views in a Single-Page Application (SPA).<br/>

Previously, creating animations for route changes required complex CSS or JavaScript logic. Now, you can apply a default transition effect instantly just by adding a single line of code to the Angular router. <br/><br/>

This API goes beyond mere aesthetics; it helps users intuitively understand the flow of your application and provides visual feedback during data loading, making the app feel faster and more responsive.

## How to Enable

Enabling the View Transitions API is incredibly simple. All you need to do is add withViewTransitions() to your provideRouter configuration in the app.config.ts file.

```jsx
// app.config.ts

import { ApplicationConfig } from '@angular/core';
import { provideRouter, withViewTransitions } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withViewTransitions() // 👈 Just add this single line!
    ),
  ],
};
```

With just this setup, a smooth cross-fade animation will be automatically applied every time the route changes.

## Applying Transitions to Specific Elements

Beyond the default fade effect, you can create animations where a specific element seamlessly transitions between two pages. For example, you can implement an effect where an image thumbnail on a list page smoothly morphs into a larger image on a detail page.

The key to this effect is the view-transition-name CSS property. By giving the same view-transition-name to the "from" and "to" elements, the browser identifies them as the same object and handles the animation automatically.

### 1. Set view-transition-name in the Template

First, dynamically assign the same view-transition-name to the corresponding elements on the list and detail pages.

**The List Page**

```jsx
<!-- books-list.component.html -->
<div class="book-grid">
  {@for (book of books; track book.id)}
    <a [routerLink]="['/book', book.id]">
      <img 
        [src]="book.cover" 
        [style.view-transition-name]="'book-cover-' + book.id"
      />
    </a>
  }
</div>
```

**The Detail Page**

```jsx
<!-- book-detail.component.html -->
{@if (book) {
  <div>
    <img 
      [src]="book.cover" 
      [style.view-transition-name]="'book-cover-' + book.id"
    />
    <h1>{{ book.title }}</h1>
  </div>
}}
```

### 2. Customizing the Transition with CSS

What happens if you only set the view-transition-name? **Even with zero CSS, the browser automatically applies a default "morphing" animation that handles the change in size and position.** This is the power of the View Transitions API.

However, you need CSS when you want to customize this default behavior—like changing the duration, easing, or adding other effects. The API exposes a set of special pseudo-elements, like ::view-transition-*, for this purpose.

For instance, if you want to change the duration and timing function of the book cover transition, you can add the following CSS:

```jsx
/* styles.css */

/* Apply to all transition groups starting with 'book-cover-' */
::view-transition-group(book-cover-*) {
  animation-duration: 0.5s; /* Set animation duration to 0.5 seconds */
  animation-timing-function: ease-in-out; /* Use a smooth easing function */
}

/* Style for the image pair (the old and new image) */
::view-transition-image-pair(book-cover-*) {
  /* Ensures the image looks good if aspect ratios differ */
  object-fit: contain;
}
```

- ::view-transition-group(name): This is the container for the entire animation of the element with the specified view-transition-name. You define the overall animation properties here. The wildcard (*) allows this rule to apply to any name starting with book-cover-.
- ::view-transition-image-pair(name): This wraps the 'old' and 'new' images being transitioned.

By leveraging CSS, you can either rely on the sensible defaults or fine-tune the animations to create an even richer user experience.

## Conclusion

The View Transitions API, now integrated into Angular 17, is a powerful feature that allows you to add a polished and dynamic user experience to your web applications with minimal code.<br/>

Starting with a single line (withViewTransitions()), you can use view-transition-name and a little bit of CSS to visually express the intuitive connection between specific elements.<br/>

This enables developers to easily build modern web apps that enhance user engagement without writing complex animation logic.

## 참고 사이트
[**View Transitions API in Angular 17**](https://medium.com/ngconf/view-transitions-api-in-angular-17-1d1ea8bb2703)