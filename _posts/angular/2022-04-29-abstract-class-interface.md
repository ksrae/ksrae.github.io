---
title: "How to create getter and setter in interface?"
date: 2022-04-29 13:13:00 +0900
comments: true
categories: angular
tags: [class, interface, getter, setter]
---

Let's delve into understanding getters and setters in classes and how to implement them with interfaces.

## Defining Getters and Setters

In object-oriented programming, classes often encapsulate data by declaring variables as `private`. Access to these variables is then controlled through methods, commonly known as getters and setters. Getters retrieve the value of a private variable, while setters modify it. The `get` and `set` keywords are used to define these special methods.<br/><br/>

Here's a TypeScript example demonstrating this concept:

```tsx
class Person {
  private _name: string;

  constructor() {
  }

  get name(): string {
    return this._name;
  }

  set name(value: string) {
    this._name = value;
  }
}

```

In this `Person` class, the `_name` variable is private. To access or modify its value, we use the `getName` and `setName` getters and setters.

## The Question: Can We Use Regular Functions Instead?

One might wonder if regular functions can achieve the same result as getters and setters:

```tsx
class Person {
  private _name: string;

  constructor() {
  }

  getName(): string {
    return this._name;
  }

  setName(value: string): void {
    this._name = value;
  }
}

```

The above code functions correctly. So, what differentiates regular functions from getters and setters?

### 1. Shared Name: Getters and Setters Can Share the Same Name

A key characteristic is that getters and setters can share the same name. While the function name is arbitrary, it's conventional to use the name of the variable.

### 2. Strict Rules: Getters and Setters Must Adhere to Specific Rules

With regular functions, you might implement something like this without issue:

```tsx
class Person {
  private _name: string;

  constructor() {
  }

  getName(postfix: string): string {
    return `${this._name}${postfix}`;
  }

  setName(value: string): string {
    this._name = value;
    return this._name;
  }
}

// It works.
```

However, if you tried to do the same with getters and setters:

```tsx
class Person {
  private _name: string;

  constructor() {
  }

  get name(postfix: string): string {
    return `${this._name}${postfix}`;
  }

  set name(value: string): string {
    this._name = value;
    return this._name;
  }
}

// error occurs.

```

Getters and setters *must* follow specific rules, and violating these rules will result in errors. This helps prevent mistakes and enforces accurate implementation. The rules are as follows:

- Getters cannot accept arguments and must return a value.
- Setters must accept one argument and cannot return a value.

### 3. Concise Syntax: Getters and Setters Allow Omission of Parentheses

When accessing getters and setters from outside the class, you can omit the parentheses.<br/>

Consider the following usage example:

``` tsx
  class Main() {
    person = new Person();

    constructor() {
      person.name = 'John'; // setter
      let name = person.name; // getter
    }

  }
```

In this example, accessing `person.name` looks like accessing a variable directly, which is only possible with getters and setters. Regular functions do not offer this syntactic sugar.

## Implementing Variables and Functions in Interfaces

Before tackling getter and setter implementation in interfaces, let's review how to implement variables and functions within interfaces.

### Variable Implementation

To implement variables in an interface, you can define their names and types:

```tsx
interface Person {
  name: string;
  age: number;
}
```

### Function Implementation

Function implementation involves defining the function signature, including parameters and return type:

```tsx
interface IPerson {
  getName: (postfix: string) => string;
  setName: (value: string) => string;
}
```

### Getter and Setter Implementation

Interestingly, interfaces do *not* directly support getter and setter definitions. The following code will result in an error:

```tsx
interface IPerson {
  get name(): string;
  set name(value: string);
}


// Applying this to a class will produce an error:
// src/app/app.component.ts: - error TS1131: Property or signature expected.
// get name(): string;
```

How can we achieve a similar effect?<br/>

The key lies in understanding the *purpose* of getters and setters: controlling access to private variables. Conversely, public variables don't necessarily *require* getters and setters.<br/><br/>

Therefore, by declaring a public variable in an interface, we can achieve the same effect as implementing a private variable with associated getters and setters. The compiler will not raise an error.<br/>

Here's how the `Person` interface can be defined:

```tsx
interface IPerson {
  name: string;
}
```

To verify this, let's implement the `IPerson` interface in the `Person` class:

```tsx
class Person implements IPerson {
  private _name: string;

  constructor(name: string) {
    this._name = name;
  }

  get name(): string {
    return this._name;
  }

  set name(value: string) {
    this._name = value;
  }
}
```

Running this code confirms that the `name` variable declared in the interface effectively acts as a substitute for getter and setter methods.<br/><br/>

This approach provides a workaround for the absence of direct getter/setter support in interfaces. There might be even better solutions, so feel free to provide feedback if you have suggestions!

