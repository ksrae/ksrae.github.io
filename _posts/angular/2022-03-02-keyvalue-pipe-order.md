---
title: "sorting of keyvalue pipe data"
date: 2022-03-02 00:00:00 +0900
comments: true
categories: angular
tags: [keyvalue, pipe]
---

Customizing Sort Order with the KeyValue Pipe in Angular Templates

Leveraging the built-in `keyvalue` pipe simplifies the process of accessing an object's keys and values within Angular templates. This article explores how to customize the sorting order when using this pipe.

# Understanding the KeyValue Pipe

The `keyvalue` pipe is a powerful tool for displaying JSON data in Angular templates, providing easy access to both keys and values. It's particularly useful with `ngFor`, allowing iteration over JSON structures without modifying the original data, separating each element into its key and value components.

## Example

Consider the following data structure in a component:

```tsx
// component
...
let data = {
  id: 1,
  name: 'test',
  age: 100,
  gender: 'unknown'
  ...
};
...
```

Using the `keyvalue` pipe, this data can be rendered as follows:

```ts
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common'; 

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule], 
  template: `
    <h2>Basic KeyValue</h2>
    <div>
      @for (item of data | keyvalue; track item.key) {
        <p>{{ item.key }} => {{ item.value }}</p>
      }
    </div>
  `,
})
export class AppComponent {
  data = {
    id: 1,
    name: 'test',
    age: 100,
    gender: 'unknown',
  };
}

/*
result:
  age => 100,
  gender => 'unknown',
  id => 1,
  name => 'test'
*/
```

Notice that the output is automatically sorted. The `keyvalue` pipe, by default, sorts the data alphabetically (a->z) by key. Let's delve into customizing this sorting behavior.

# Implementing Custom Sorting

To sort the data in a specific order using the `keyvalue` pipe, it's essential to understand the underlying sorting mechanism. Before diving into `keyvalue`, let's review basic JavaScript sorting.

## Basic Sorting in JavaScript

The JavaScript `sort()` function, when used without a custom comparison function, sorts elements alphabetically. To modify this behavior, a comparison function can be provided.

For example, to sort an array numerically in ascending order:


```tsx
arr.sort((a, b) => a - b);
```

Conversely, to sort in descending order:

```tsx
arr.sort((a, b) => b - a);
```


## Sorting with KeyValue Pipe

The `keyvalue` pipe allows for a custom comparison function to be passed as a parameter, enabling tailored sorting.

```
{% raw %}{{ input_expression | keyvalue [ : compareFn ] }}{% endraw %}

compareFn: (a: KeyValue<K, V>, b: KeyValue<K, V>)
```

By default, the `keyvalue` pipe uses an ascending sorting function if no parameter is provided. To implement custom sorting, define a comparison function and pass it to the `keyvalue` pipe.<br/>
<br/>
Consider the following example:

```tsx
// component
...
let data = {
  id: 1,
  name: 'test',
  age: 100,
  gender: 'unknown'
  ...
};

// ascend
ascending = (a: KeyValue<K, V>, b: KeyValue<K, V>) => {
  return a - b;
}

original = (a: KeyValue<K, V>, b: KeyValue<K, V>) => {
  return 0;
}

descending = (a: KeyValue<K, V>, b: KeyValue<K, V>) => {
  return b - a;
}
...
```

```html
  @for (item of data | keyvalue: originalOrder; track item.key) {
    <p>{{ item.key }} => {{ item.value }}</p>
  }

<!-- result:
  id => 1,
  name => 'test',
  age => 100,
  gender => 'unknown'
-->
```

In this scenario, the `original` function, which returns 0, effectively disables sorting, preserving the original order of the JSON data.  You can replace `original` with `ascending` or `descending` (assuming the keys are numbers) or your own custom comparison function to achieve the desired sort order.


# References

- [Angular Official Documentation for KeyValuePipe](https://angular.io/api/common/KeyValuePipe)
- [Stack Overflow: angular-keyvalue-pipe-sort-properties-iterate-in-order](https://stackoverflow.com/questions/52793944/angular-keyvalue-pipe-sort-properties-iterate-in-order)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/keyvalue-pipe%EB%A1%9C-data-%EC%A0%95%EB%A0%AC%ED%95%98%EA%B8%B0)
<br/>

