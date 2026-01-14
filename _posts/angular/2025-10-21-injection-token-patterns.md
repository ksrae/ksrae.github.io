---
title: Injection Token Patterns
date: 2025-10-21 12:00:00 +0900
comments: true
categories: angular
tags: [inject, di]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Angular-%EA%B3%A0%EA%B8%89-DI-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0-InjectionToken%EC%9D%98-%EB%AA%85%EC%8B%9C%EC%A0%81-%EC%A3%BC%EC%9E%85%EA%B3%BC-%EC%9E%90%EB%8F%99-%EC%A3%BC%EC%9E%85-%ED%8C%A8%ED%84%B4-%EB%B9%84%EA%B5%90)
<br/>

**Title: Mastering Advanced Angular DI: Explicit vs. Automatic InjectionToken Patterns**

Angular's Dependency Injection (DI) system offers capabilities far beyond the basic use of the @Injectable decorator. By leveraging InjectionToken, you can achieve the highest level of decoupling, completely separating a class's implementation from its consumption.

This article provides a comparative analysis of two powerful patterns for making a class without @Injectable injectable using InjectionToken: one where the consumer must explicitly register a Provider, and another where the token provides for itself, requiring zero configuration from the consumer.

### The Foundation: Defining the Class to be Injected

Before comparing the two patterns, let's define the plain TypeScript class we want to inject. This class internally uses inject() to depend on HttpClient.

**data-processor.ts**

```jsx
import { HttpClient } from '@angular/common/http';
import { inject } from '@angular/core';
import { Observable } from 'rxjs';

// A plain class without @Injectable, common to both patterns.
export class DataProcessor {
  private http = inject(HttpClient);
  private endpoint = 'https://jsonplaceholder.typicode.com/todos/1';

  fetch(): Observable<any> {
    console.log('Fetching data from DataProcessor...');
    return this.http.get(this.endpoint);
  }
}
```

### Pattern 1: Manual Registration with an Explicit Provider

This pattern emphasizes "Separation of Concerns." The class file provides an injection "recipe" (the Provider), and the application's configuration file (app.config.ts) explicitly decides whether to use that recipe and registers it.

### 1. Defining the Token and Provider (data-processor-explicit.ts)

```jsx
import { InjectionToken, Provider } from '@angular/core';
import { DataProcessor } from './data-processor'; // Import the common class

// 1. The unique key (Token) for the DI system
export const DATA_PROCESSOR_TOKEN = new InjectionToken<DataProcessor>('DataProcessor');

// 2. The recipe (Provider) that defines the injection method
export const DATA_PROCESSOR_PROVIDER: Provider = {
  provide: DATA_PROCESSOR_TOKEN,
  useClass: DataProcessor
};
```

This file exports both DATA_PROCESSOR_TOKEN and DATA_PROCESSOR_PROVIDER.

### 2. Registering the Provider in app.config.ts

For the application to use this service, it must manually add DATA_PROCESSOR_PROVIDER to the providers array.

**app.config.ts (Pattern 1)**

```jsx
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
// Import the Provider directly.
import { DATA_PROCESSOR_PROVIDER } from './data-processor-explicit';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    
    // Explicitly registering the Provider is mandatory.
    DATA_PROCESSOR_PROVIDER,
  ],
};
```

### Pattern 2: Automatic Registration with providedIn: 'root'

This pattern aims for "maximum convenience" and "perfect encapsulation." The token itself knows how to register itself with the root injector, so the consumer requires zero DI setup. It shares the same philosophy as @Injectable({ providedIn: 'root' }).

### 1. Defining a Self-Providing Token (data-processor-auto.ts)

```jsx
import { InjectionToken } from '@angular/core';
import { DataProcessor } from './data-processor'; // Import the common class

// Define all Provider info within the Token's constructor.
export const DATA_PROCESSOR_TOKEN_AUTO = new InjectionToken<DataProcessor>(
  'DataProcessor Auto',
  {
    // ⭐️ This ensures the token is automatically registered in the root injector.
    providedIn: 'root',
    
    // ⭐️ The factory function to execute when this token is requested.
    factory: () => new DataProcessor(),
  }
);
```

No separate Provider object is needed. This file now exports only DATA_PROCESSOR_TOKEN_AUTO.

### 2. app.config.ts - No Configuration Needed!

When using this pattern, you don't need to add any configuration related to DataProcessor in app.config.ts.

**app.config.ts (Pattern 2)**

```jsx
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    // No configuration for DataProcessor is needed here.
  ],
};
```

### Component Usage: Identical for Both Patterns

Surprisingly, the code in the consuming component is identical regardless of which pattern you choose. The only difference is which token is imported.

**app.component.ts**

```jsx
import { Component, inject } from '@angular/core';
// If using Pattern 1:
// import { DATA_PROCESSOR_TOKEN } from './data-processor-explicit';
// If using Pattern 2:
import { DATA_PROCESSOR_TOKEN_AUTO } from './data-processor-auto';

@Component({ ... })
export class AppComponent {
  // The injection code is the same for any token.
  private processor = inject(DATA_PROCESSOR_TOKEN_AUTO); // or DATA_PROCESSOR_TOKEN

  constructor() {
    this.processor.fetch().subscribe(console.log);
  }
}
```

### When to Use Which Pattern?

| **Feature** | **Pattern 1 (Explicit Provider)** | **Pattern 2 (Self-Providing Token)** |
| --- | --- | --- |
| **Configuration** | Consumer **manually registers** Provider in app.config.ts | **Automatic registration** via providedIn: 'root' (no config needed) |
| **Coupling** | Relatively higher (config file must know about the Provider) | **Minimized** (consumer only needs to know the token) |
| **Explicitness** | **Very high**. One look at app.config.ts shows what's provided. | Relatively lower (can feel like "magic") |
| **Primary Use Case** | - Centrally managing all DI configuration at the app's top level.<br>- Easily swapping implementations (useClass, useValue) for different environments. | - Creating **reusable libraries**.<br>- Making modules or services that should "just work" out of the box. |

### Conclusion

Both patterns are powerful ways to make a class injectable without using the @Injectable decorator.

- The **Explicit Provider pattern** is advantageous when you want to centralize all DI configuration, offering clarity and making the code's flow easy to follow.
- The **Self-Providing Token pattern** offers the ultimate in decoupling and reusability, making it an essential technique, especially for library authors.

By choosing the appropriate pattern based on your project's architecture and requirements, you can build cleaner, more flexible, and more scalable Angular applications.