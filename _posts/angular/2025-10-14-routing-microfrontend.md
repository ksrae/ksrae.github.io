---
title: Taming the Router in Angular Micro-Frontends"
date: 2025-10-14 12:00:00 +0900
comments: true
categories: angular
tags: [microfrontend, location, routing]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C-%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C%EC%97%90%EC%84%9C-%EB%9D%BC%EC%9A%B0%ED%8C%85-%EA%B4%80%EB%A6%AC)
<br/>

## Taming the Router in Angular Micro-Frontends: A Guide to Independent Popup Navigation

When managing multiple applications in an Angular Micro-Frontend (MFE) environment, you can often run into unexpected challenges. One of the most common is **Router Management**.

In this article, we'll explore a fascinating problem and its solution: how to display another Angular app as a popup over a main application and have it handle its own routing independently.

## What's the Problem? Two Homes, One Address

Here's the situation we faced: A user is on the main application (`http://my-app.com/dashboard`) and clicks a button to open a popup for filling out a simple form. This popup is, in fact, a separate Micro-Frontend app built as its own Angular project.

The challenge arises because this popup app also contains multiple pages (`/form/step1, /form/step2`) and needs to navigate between them freely.

What would happen if the popup app's router behaved as usual? Every time the user navigates within the popup, the browser's main URL would change to something like `http://my-app.com/form/step2`. This triggers an unwanted full-page navigation, causing the main app's dashboard page behind it to disappear.

**Our Goal:** To allow the popup app to navigate internally as if its URL is changing, while keeping the main app's browser URL completely unchanged!

## The Key Insight: How the Angular Router Works

To solve this, we need to understand how the Angular Router communicates with the browser. Let's use a simple analogy:

- **Router**: The Navigation App (Destination: /form/step2)
- **Location / LocationStrategy**: The Driver (Takes instructions from the navigation app to actually turn the wheel and press the pedals)
- **Browser APIs**: The actual Car and Road (The car moves and the street address changes based on the driver's actions)

Normally, when the Router issues a command like "Go to `/form/step2`!", the LocationStrategy takes that instruction and directly changes the browser's address bar. Our task is to replace this "driver" with one who operates a simulator (a virtual environment) instead of driving the real car.

## The Core Tools: Rediscovering Angular's Testing Library

Surprisingly, the solution to this problem was hiding within Angular's testing utilities: MockLocationStrategy and SpyLocation.

### 1. What is MockLocationStrategy?

As its name suggests, this is a "mock" LocationStrategy. It's originally intended for use in test environments to verify that routing logic works correctly without actually changing the browser's state.

This "fake driver" receives the command "Go to `/form/step2`!" from the Router but **never touches the browser's steering wheel**. Instead, it simply records in its own memory, "My current location is `/form/step2`." The browser's URL remains completely unaffected.

### 2. What is SpyLocation?

This acts like a "spy" that listens for location changes. When MockLocationStrategy changes its internal path, SpyLocation can detect that change and emit an event, effectively announcing, "Hey, the internal path just changed to `/form/step2`!" This allows the main application to be aware of what's happening inside the popup app if needed.

## The Implementation: Swapping Out the "Driver" in Our Popup App

Now, let's modify the configuration of our Micro-Frontend app that will be displayed as a popup. Using Angular's Dependency Injection (DI) system, we can easily swap the default "driver" with our "fake driver."

In the popup app's main.ts or bootstrapApplication config, add the following providers:

```tsx
// main.ts of the Angular app to be used as a popup
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { Location, LocationStrategy, SpyLocation } from '@angular/common';
import { MockLocationStrategy } from '@angular/common/testing'; // 👈 Imported from the testing module
import { AppComponent } from './app/app.component';
import { routes } from './app/app.routes';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),

    // 👇 Must be defined!
    { provide: Location, useClass: SpyLocation },
    { provide: LocationStrategy, useClass: MockLocationStrategy }
  ]
});`
```

### How does this providers configuration work?

- `{ provide: LocationStrategy, useClass: MockLocationStrategy }`: This line tells Angular's DI system: "Whenever any part of this application asks for the LocationStrategy (the default driver), give it our MockLocationStrategy (the fake driver) instead."
- `{ provide: Location, useClass: SpyLocation }`: Similarly, when the Location service is requested, we provide SpyLocation to enable detection of internal path changes.

With this setup, the popup application now has its own virtual routing world, completely decoupled from the browser's address bar.

## Putting It All Together: The Step-by-Step Flow

1. A user clicks a link inside the popup app, such as `<a routerLink="/form/step2">`.
2. The popup app's Router requests a path change from its LocationStrategy.
3. Because we've configured MockLocationStrategy as the "driver," this request is handled entirely in memory and is **not** passed on to the browser.
4. SpyLocation detects this internal path change.
5. **Result:** The browser's URL remains `http://my-app.com/dashboard`, while the popup app successfully renders the `/form/step2` component on screen.

## Conclusion

By leveraging our understanding of the Angular Router's internal mechanics and creatively applying tools originally designed for testing, we were able to solve the complex problem of managing multiple, independent routers on a single page.

This approach can be a powerful solution for maintaining a seamless user experience while ensuring the independence of each app in a micro-frontend architecture.

Of course, using a testing library in a production environment warrants some consideration regarding long-term maintenance, but it stands as an excellent example of Angular's flexibility and extensibility.

Use Arrow Up and Arrow Down to select a turn, Enter to jump to it, and Escape to return to the chat.