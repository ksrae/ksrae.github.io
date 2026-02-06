---
title: "Handling Multiple Observables in @if"
date: 2021-06-30 14:16:00 +0900
comments: true
categories: angular
tags: [ngif, observable]
---

## Plan

1. Define a single Observable within `ngIf or @if`.
2. Define multiple Observables within `ngIf or @if`.

## Applying a Single Observable

To utilize the `async` pipe, append it to the variable name and define a variable name to use with `as`. This allows direct access to the resolved value of the Observable within the template.

```html
<!-- control flow -->
@if (user$ | async; as user) {
  <div>
    <p>{{ user.name }}</p>
  </div>
}
```

```html
<!-- ngif -->
{% raw %}
<div *ngIf="user$ | async as user">
  <p>{{user.name}}</p>
</div>
{% endraw %}
```

## Applying Multiple Observables

Defining multiple Observables in `@if or ngIf` in the same manner as a single Observable will result in an error. This is due to the template parser's inability to correctly handle the combined asynchronous operations directly within the `@if or *ngIf` directive.

```html
<!-- control flow -->
@if ((user$ | async; as user) || (item$ | async; as item)) {
  <!-- This syntax is not valid -->
}
```

```html
<!-- ngif -->
{% raw %}
<!-- error -->
<div *ngIf="(user$ | async as user) || (item$ | async as item)"></div>
{% endraw %}
```

## Solution - Applying Multiple Observables

To work with multiple Observables effectively, transform them into an object. This approach allows the `@if or *ngIf` directive to manage the asynchronous subscriptions properly. The code below demonstrates the correct implementation and will execute without errors. This encapsulates multiple Observables into a single object that can be evaluated.

```html
<!-- control flow -->
@if ({ user: user$ | async, item: item$ | async }; as data) {
  <div>
    @if (data.user; as user) {
      <p>User Name: {{ user.name }}</p>
    }

    @if (data.item; as item) {
      <p>Item Name: {{ item.name }}</p>
    }
  </div>
}
```

```html
<!-- ngif -->
{% raw %}
<div *ngIf="{user: user$ | async, item: item$ | async} as data">
	<p>{{data.user.name}}</p>
  <p>{{data.item.name}}</p>
</div>
{% endraw %}
```

## Reference

- [ngx-translate-router](https://github.com/gilsdav/ngx-translate-router)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Control-Flow%EC%97%90%EC%84%9C-%EC%97%AC%EB%9F%AC-Observable-%EB%8B%A4%EB%A3%A8%EA%B8%B0)
<br/>