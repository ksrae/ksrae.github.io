---
title: "Understanding of Standalone and importance of main.ts"
date: 2024-09-05 22:04:00 +0900
comments: true
categories: angular
tags: [standalone, maints]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Standalone%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-main.ts%EC%9D%98-%EC%A4%91%EC%9A%94%EC%84%B1)
<br/>

Angular has introduced several features aimed at simplifying application architecture and optimizing performance. Among these, the most significant is the Standalone component, a pivotal feature that has fundamentally altered the way Angular applications are developed. By moving away from the traditional NgModule-based architecture, it allows components to be declared and used independently, minimizing module dependencies and streamlining the application initialization process. This article explores the concept of Angular Standalone components, why you should use them, and the necessity of initial loading tasks in `main.ts`.

# Understanding Standalone Components

Traditionally, Angular applications operated on the foundation of NgModules. Each feature was defined within one or more modules, with components, services, directives, and more all declared as part of a module. However, this approach could lead to excessive complexity, especially in smaller applications, and the need to manage modules alongside components or services when reusing them was inconvenient.<br/><br/>

To address these issues, Angular introduced the concept of Standalone components. Standalone components can be independently defined without the need for a traditional module, reducing unnecessary module declarations and simplifying the application's structure. When using Standalone components, you can directly declare the modules, directives, and pipes that a component needs in its `imports`. Consequently, this allows for the creation of lighter and faster applications.

```tsx
@Component({
  standalone: true,
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
  imports: [CommonModule, FormsModule] // Directly import required modules
})
export class AppComponent { }
```

# The Role of `main.ts`

In Angular, the `main.ts` file serves as the application's entry point. This file is responsible for bootstrapping the application in the browser and handles the initial settings required when the application is first executed. In the traditional Angular approach, it was used to bootstrap the AppModule, but with the introduction of Standalone components, you can directly bootstrap a root component like AppComponent without a module.<br/><br/>

During this process, `main.ts` is responsible for essential initial settings, particularly defining routing, HTTP client configuration, and internationalization (i18n) functionalities. In other words, while the necessity of modules has decreased, each feature must be directly loaded in `main.ts` and declared as a provider to perform global application settings.

```tsx
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch(err => console.error(err));
```

In the code above, `bootstrapApplication` sets AppComponent as the application's root component and loads the application in the browser.

# Initial Loading Tasks in `main.ts`

The initial loading tasks performed in `main.ts` play a crucial role in controlling the application's behavior. Specifically, setting up the HTTP client, routing, and handling internationalization should be done in this file. In the Standalone component approach, these settings are passed through providers instead of modules, and Angular uses them as global services.

## Providers

### HTTP Client Configuration

Angular's HttpClient is a core module responsible for communication with servers. In Standalone applications, HttpClientModule can still be used for server communication, and it can be initialized in `main.ts`. For example, settings for XSRF protection, JSONP support, etc., can be declared in `main.ts`.

```tsx
bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(
      withJsonpSupport(),
      withXsrfConfiguration({
        cookieName: 'XSRF-TOKEN',
        headerName: 'X-XSRF-TOKEN'
      }),
      withNoXsrfProtection()
    )
  ]
});
```

#### withJsonpSupport()

JSONP is mainly used to solve CORS issues. When requesting resources from a different domain, if the server does not have CORS settings configured, JSONP can be used to receive responses. `withJsonpSupport()` adds JSONP support to Angular's HttpClient. This should only be enabled when necessary.

#### withXsrfConfiguration()

This option configures settings for XSRF (Cross-Site Request Forgery) protection. By default, Angular supports XSRF protection using cookies and headers. This function allows you to customize the cookie name and header name used for protection.

- `cookieName: 'TOKEN'`: Sets the name of the cookie containing the CSRF token. The default value is `XSRF-TOKEN`.
- `headerName: 'X-TOKEN'`: Sets the name of the header that includes the XSRF token sent to the server upon request. The default value is `X-XSRF-TOKEN`.

#### withNoXsrfProtection()

This option disables XSRF protection. It should not be used for security-sensitive API calls and can be used in situations where XSRF tokens do not need to be sent. For example, it can be used when communicating with external APIs.

## Zoneless: A New Approach to Change Detection

### Concepts and Changes in Zoneless

Previously, the zone.js library was used to automatically trigger Change Detection. However, zone.js had drawbacks in terms of performance and development experience. To solve these issues, Angular introduced a new approach called Zoneless starting from version 18. Zoneless is a method that does not rely on zone.js, providing performance optimization and various other benefits.

Using Zoneless provides faster initial rendering and runtime, smaller bundle sizes, and a simplified debugging experience. To use the Zoneless feature, add `provideExperimentalZonelessChangeDetection` when bootstrapping the application and remove zone.js from the `angular.json` file.

```tsx

bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});
```

In the Zoneless approach, most components that use the existing `ChangeDetectionStrategy.OnPush` are compatible, and UI updates are made using state management with signals.

```tsx
@Component({
  standalone: true,
  template: `
    <h1>Hello from {{ name() }}!</h1>
    <button (click)="handleClick()">Go Zoneless</button>
  `,
})
export class AppComponent {
  protected name = signal('Angular');

  handleClick() {
    this.name.set('Zoneless Angular');
  }
}
```

In this approach, Angular does not automatically trigger Change Detection when a state change occurs; instead, the UI is only changed when a signal is updated. Additionally, a new scheduler combines consecutive Change Detections to provide optimized performance.

### Advantages of Zoneless

- Improved Interoperability with Micro-Frontends and Other Frameworks: Zoneless is more flexible when used with other frameworks.
- Faster Initial Rendering and Runtime: Performance is improved by removing unnecessary zone.js triggers.
- Reduced Bundle Size: The zone.js removal allows for maintaining a smaller bundle size.
- Simpler Debugging Experience: Stack traces become more concise and clear.
- Simpler Debugging and Maintenance: Debugging becomes more intuitive, and issues where Change Detection is triggered multiple times due to scheduling can be resolved.

### How to Transition to Zoneless

Transitioning to Zoneless is relatively simple, and components that previously used `ChangeDetectionStrategy.OnPush` can be transitioned without significant modifications. You can also enable the event merging feature by using `provideZoneChangeDetection` in `bootstrapApplication`.

```tsx
bootstrapApplication(AppComponent, {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true })
  ]
});
```

Additionally, native async/await can be used in Zoneless applications, eliminating the need for downleveling that was required with zone.js. This improves bundle size and debugging performance.

## `provideRouter` Routing Configuration

Angular routing configuration is also performed in `main.ts`. In the Standalone approach, use `provideRouter` instead of RouterModule to provide routing settings. This allows for controlling navigation behavior through various additional settings along with route configurations.

```
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(
      appRoutes,
      withPreloading(PreloadAllModules),
      withDebugTracing(),
      withDisabledInitialNavigation(),
      withEnabledBlockingInitialNavigation(),
      withInMemoryScrolling(),
      withRouterConfig(),
    )
  ]
});
```

### withPreloading()

This option is used to specify which modules to preload.

- `NoPreloading`: Does not preload modules and only loads them when the user navigates to that route.
- `PreloadAllModules`: Preloads all modules that may be needed later after the application is loaded. Useful for improving page loading speed in large applications.

### withDebugTracing()

Using this option allows you to debug by printing various events that occur while the Angular router is running to the console. Useful when you want to track the routing flow.

### withDisabledInitialNavigation()

Disables automatic navigation when the router is first loaded after the application bootstrap. It can be used in applications with complex initialization logic or timing issues.

### withEnabledBlockingInitialNavigation()

An option to set the application to wait until navigation is complete before the bootstrap is completed. This option is essential for server-side rendering (SSR). Using this option delays the app loading until the initial navigation is complete.

### withInMemoryScrolling()

This option controls scrolling behavior. For example, you can define behaviors such as maintaining the previous scroll position when moving pages or moving to the top of the page.

- `anchorScrolling: 'enabled'`: Enables scrolling via anchors.
- `scrollPositionRestoration: 'enabled' | 'top' | 'disabled'`: Sets how to restore the scroll position after moving pages.
- `'enabled'`: Restores the scroll position of the previous page.
- `'top'`: Scrolls to the top of the newly loaded page.
- `'disabled'`: Does not restore the scroll position.

### withRouterConfig()

This option allows for more fine-grained adjustment of the router's overall behavior.

- `canceledNavigationResolution: 'replace' | 'computed'`: Determines how to handle the URL if navigation is canceled.
- `onSameUrlNavigation: 'reload' | 'ignore'`: Determines how to handle navigation to the same URL. 'reload' reloads the URL, and 'ignore' ignores the navigation.
- `paramsInheritanceStrategy: 'always' | 'emptyOnly'`: Sets how child routes inherit parameters from the parent. 'always' always inherits, and 'emptyOnly' only inherits when the child route has no parameters.
- `urlUpdateStrategy: 'deferred' | 'eager'`: Determines when the URL is updated. 'deferred' updates the URL after navigation is complete, and 'eager' updates the URL immediately when navigation starts.

## `importProvidersFrom` Module Calls

`importProvidersFrom` helps you easily integrate existing Angular modules used in module-based systems into standalone applications. This allows you to import existing modules. Representative items that can be passed to `importProvidersFrom` are as follows:

- `HttpClientModule`: A module that supports HTTP communication. Provides the functionality needed to handle HTTP requests and responses.
- `BrowserModule`: An essential module needed to run the application in the browser. Usually only imported once in the root module.
- `TranslateModule.forRoot()`: A module for internationalization support that defines the settings for language translation. In the example, it contains logic for loading translation files along with TranslateLoader.

You can add other necessary Angular modules besides these. For example, you can include FormsModule and ReactiveFormsModule to handle forms or set up routing with RouterModule.

```tsx
bootstrapApplication(AppComponent, {
  providers: [
    importProvidersFrom([
      HttpClientModule,
      TranslateModule.forRoot({
        loader: {
          provide: TranslateLoader,
          useFactory: (http: HttpClient) => new TranslateHttpLoader(http, './assets/i18n/', '.json'),
          deps: [HttpClient]
        }
      })
    ])
  ]
});
```

# Conclusion: Advantages of the Standalone Approach and the Importance of `main.ts`

Standalone components provide a more flexible and simpler development approach, moving away from the traditional NgModule-based architecture. This allows you to optimize the application structure and remove unnecessary complexity. However, when using the Standalone approach, you must carefully configure the initialization tasks in `main.ts`, especially since global settings (e.g., HTTP client, routing, internationalization, etc.) are all defined in this file.

This approach allows you to develop and use components independently without relying on modules, which is especially helpful in reducing the complexity of module management in large applications. Angular's Standalone components and the `main.ts` configuration method that supports them will change the way Angular applications are developed in a more efficient and intuitive manner in the future.