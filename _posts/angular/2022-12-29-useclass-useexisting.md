---
title: "different between useClass and useExisting"
date: 2022-12-29 11:52:00 +0900
comments: true
categories: angular
tags: [provider, useexisting, useclass]
---

[한국어(Korean) Page](https://velog.io/@ksrae/useClass%EC%99%80-useExisting%EC%9D%98-%EB%B9%84%EA%B5%90)
<br/>

In this post, we'll explore the difference between useClass and useExisting when configuring providers in Angular's Dependency Injection (DI) system. This concept applies equally to both NgModules and Standalone Components.

## useClass
The useClass provider recipe tells the injector to return a new instance of a specified class when a particular token is requested. In other words, when you ask for token A, the injector creates and provides a fresh instance of class B.
Here's how to use it. You can configure this in the providers array of either a Standalone Component or an NgModule.

```ts
// In a Standalone Component's providers array
@Component({
  ...
  standalone: true,
  providers: [
    { provide: ProductService, useClass: FakeProductService }
  ]
})

// In an NgModule's providers array
@NgModule({
  ...
  providers: [
    { provide: ProductService, useClass: FakeProductService }
  ]
})
```

## useExisting
The useExisting provider recipe creates an alias for an existing token. When you ask for token A, the injector returns the existing instance that was already created for another token (B).
To use this approach, the token you are referencing (B) must already be registered in the providers array.

```ts
// In a Standalone Component's providers array
@Component({
  ...
  standalone: true,
  providers: [
     NewProductService,
     { provide: ProductService, useExisting: NewProductService },
  ]
})

// In an NgModule's providers array
@NgModule({
  ...
  providers: [
     NewProductService,
     { provide: ProductService, useExisting: NewProductService },
  ]
})
```

## Difference between useClass and useExisting
At first glance, useClass and useExisting may seem similar, but there's a critical difference: whether a new instance is created or an existing one is shared.
Let's dive into an example to understand this difference clearly.
Setting Up Services for the Example
Let's assume we have two services. They have a similar shape, but their add methods behave differently so we can easily tell them apart.

```ts
// test.service.ts
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class TestService {
  index = 0;

  add(): number {
    return ++this.index;
  }
}
```


```ts
// new-test.service.ts
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class NewTestService {
  index = 0;

  add(): number {
    this.index += 10;
    return this.index;
  }
}
```

Now, let's create a Standalone Component that injects these two services and calls their methods every second.
code

```ts
// app.component.ts
import { Component } from '@angular/core';
import { TestService } from './test.service';
import { NewTestService } from './new-test.service';

@Component({
  selector: 'app-root',
  standalone: true,
  template: `<h1>useClass vs useExisting</h1>`,
  // The providers configuration will change in the following scenarios.
})
export class AppComponent {
   constructor(
      private testService: TestService,
      private newTestService: NewTestService
   ) {
     setInterval(() => {
        console.log('test     ->', this.testService.add());
        console.log('newTest  ->', this.newTestService.add());
        console.log('---');
     }, 1000);
   }
}
```

We will now change the providers configuration in AppComponent to see how the behavior changes.

### 1. Default Provider Configuration
First, let's use the most common approach where each service is provided individually. Since both services are registered globally with providedIn: 'root', the result is the same even if the component's providers array is empty.

```ts
// app.component.ts
@Component({
  ...
  // If the providers array is empty, dependencies are resolved from the root injector.
  // To be explicit, the configuration would look like this:
  providers: [TestService, NewTestService] 
})
export class AppComponent { ... }
```

#### Result:
As expected, two separate instances are created, and each service operates independently.

```
// result
test     -> 1
newTest  -> 10
---
test     -> 2
newTest  -> 20
---
test     -> 3
newTest  -> 30
---
```

### 2. Using useClass
Now, let's use useClass to provide an instance of TestService whenever the NewTestService token is requested.

```ts
// app.component.ts
@Component({
  ...
  providers: [
    TestService,
    { provide: NewTestService, useClass: TestService },
  ]
})
export class AppComponent { ... }
```

When testService is injected: An instance of TestService is created.
When newTestService is injected: The provider is configured with { provide: NewTestService, useClass: TestService }, so a new instance of TestService is created.

#### Result:
newTestService behaves like TestService, but it is a different instance from testService. Therefore, they do not share state.

``
// result
test     -> 1
newTest  -> 1
---
test     -> 2
newTest  -> 2
---
test     -> 3
newTest  -> 3
---
```

### 3. Using useExisting
Finally, let's replace useClass with useExisting under the same conditions.

```ts
// app.component.ts
@Component({
  ...
  providers: [
    TestService,
    { provide: NewTestService, useExisting: TestService },
  ]
})
export class AppComponent { ... }
```

When testService is injected: An instance of TestService is created.
When newTestService is injected: The provider is configured with { provide: NewTestService, useExisting: TestService }, so the injector returns the already existing instance that was created for the TestService token.

#### Result:
newTestService and testService point to the exact same instance. As a result, they share their state (the index value).

```
// result
test     -> 1
newTest  -> 2
---
test     -> 3
newTest  -> 4
---
test     -> 5
newTest  -> 6
---
```

## Conclusion
- useClass: "When this token is requested, create a new instance of that class."
Even if two tokens use the same class, they will get separate instances.
- useExisting: "When this token is requested, give me the existing instance that is already associated with that other token."
This allows two tokens to share a single instance, acting as an alias.
Understanding this distinction allows you to leverage Angular's Dependency Injection system with greater flexibility and power.