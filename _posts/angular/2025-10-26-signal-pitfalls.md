---
title: tsconfig setting in Angular
date: 2025-10-26 12:00:00 +0900
comments: true
categories: angular
tags: [signal]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Signal%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-%EC%82%AC%EC%9A%A9%EB%B0%A9%EB%B2%95)
<br/>


## Angular Signals: A Deep Dive into Core Principles and Common Pitfalls

Since Angular 17, Signals have become the framework's core reactivity model. While many developers' first impression is that they are "easier than RxJS," they often encounter unexpected behavior when applying them in real-world projects. This is because Signals operate on a fundamentally different principle than traditional reactive programming libraries like RxJS.

In this article, we'll analyze the root causes of this common confusion, dive into the core mechanics of how Signals work, and establish the correct patterns for using them effectively.

### What's the Problem? - A Clash of Mental Models

Here's a situation you've likely faced: you declare a Signal to manage a component's state and update its value within some logic. However, you find that while some parts of your template update, others remain completely static.

The problem stems from our **existing mental models**, where we tend to think of Signals as an extension of RxJS Observables or just plain mutable variables.

- **Observable (Stream):** A **waterfall** of data that emits multiple values over time. You must connect to this flow by subscribing.
- **Signal (State Container):** A **cup of water** that holds a single value at a specific point in time. When the value changes, it simply announces, "the water has changed." There is no concept of a continuous flow.

Without recognizing this fundamental difference, it's difficult to understand why your code isn't behaving as expected.

**Our Goal:** To understand how Signals interact with Angular's change detection system, clearly differentiate the roles of computed and effect, and write predictable, bug-free code.

### The Key Insight: 3 Core Principles of How Signals Work

To solve this, we must understand that a Signal is not just a 'value' but a tool that operates within the Angular ecosystem according to specific rules.

### 1. Signals Do Not Bypass Change Detection

One of the biggest misconceptions is the belief that "if I just change a Signal's value, Angular will magically update everything." In reality, Signals **do not replace** Angular's existing change detection system; they are a tool to make it **more efficient**.

- **Template Binding:** When a Signal is bound directly in a template (e.g., {{ mySignal() }}), Angular is smart enough to optimize rendering, re-running change detection only for the specific component or view that needs it.
- **Unconnected Logic:** If you change a Signal's value inside a regular function or logic that isn't connected to the template, Angular is under no obligation to detect that change and update the view.

### 2. computed is 'Derived State', effect is a 'Side Effect'

Using these two functions interchangeably only leads to more confusion. Their roles are completely different.

- **computed**: A **blueprint or calculated formula**. It creates a new 'read-only' Signal that depends on one or more other Signals. It must be pure, is memoized (cached), and only recalculates when one of its dependencies changes. **Use it to create new state values.**
- **effect**: An **execution command or notification**. It 'runs' a piece of code whenever a Signal it depends on changes. It does not return a value. **Use it for side effects that impact the outside world**, like logging, saving data, or direct DOM manipulation.

### 3. Changes are Detected by Reference

Signals do not detect when an internal property of an object or an element of an array changes. They only recognize a change when the value is replaced with a **new reference**.

- **set(value)**: Replaces the value with a completely new reference.
- **update(fn)**: Takes the previous value and returns a new value to replace it.

This principle reinforces why practicing immutability is so crucial when working with Signals.

### Practical Implementation: Common Mistakes and Correct Usage

With these principles in mind, let's look at common mistakes found in real-world code.

### Case 1: The UI Doesn't Update After Adding to an Array

```jsx
const todos = signal<string[]>(['Study']);

// ❌ WRONG: The Signal won't detect a change in reference
function addTodoWrong(newTodo: string) {
  todos().push(newTodo); // Mutates the internal array, but the reference stays the same
}

// ✅ RIGHT: Create a new array to replace the old reference
function addTodoRight(newTodo: string) {
  todos.update(currentTodos => [...currentTodos, newTodo]);
}
```

**Why?** By using the spread operator (...) in the update callback, we return a brand new array. Because the reference has changed, the todos Signal knows its value has been updated and triggers a UI refresh.

### Case 2: Using effect When You Really Need computed

```jsx
const firstName = signal('John');
const lastName = signal('Doe');
let fullName = ''; // A plain variable

// ❌ WRONG: fullName is not reactive, and effect returns no value
effect(() => {
  fullName = `${firstName()} ${lastName()}`;
  console.log('Name changed:', fullName);
});

// ✅ RIGHT: Use computed for derived state
const fullNameSignal = computed(() => `${firstName()} ${lastName()}`);

// Now you can use {{ fullNameSignal() }} in your template
```

**Why?** computed creates a new reactive value, fullNameSignal, which can be reused in templates or other Signals. An effect is a dead-end; it simply performs an action and does not produce a usable state.

## Conclusion

Angular Signals are not a replacement for RxJS. They are a **new paradigm for managing component state intuitively and efficiently**. The key is to abandon old mental models and embrace the core principles of how Signals work.

- Signals are **state containers**, not streams.
- Signals **work in concert with** Angular's change detection system.
- Changes are only detected when you **replace the reference**, enforcing immutability.

Just as we solved the complex router problem in a micro-frontend architecture, we can leverage these principles to build more predictable and stable applications with Signals. They are not a silver bullet, but when used in the right context, they are one of the most powerful tools in your Angular toolbox.

## Reference
[The Most Misunderstood Concept in Angular Signals](https://medium.com/@julias3/the-most-misunderstood-concept-in-angular-signals-1929f5c27c13)