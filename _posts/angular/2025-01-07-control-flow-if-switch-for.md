---
title: "Control flow - If, Switch and For"
date: 2025-01-07 10:41:00 +0900
comments: true
categories: angular
tags: [controlflow, ngif, ngfor]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Control-flow-%EC%83%88%EB%A1%9C%EC%9A%B4-If-Switch-%EA%B7%B8%EB%A6%AC%EA%B3%A0-For)
<br/>

Angular is a powerful framework that offers various features to help developers build web applications efficiently. One of these features is Control Flow. In this article, we'll delve into what Control Flow is, how it's used, and the benefits it brings to Angular development.

# Understanding Angular Control Flow

Control Flow provides a more intuitive and concise way to write conditional statements and loops within Angular templates. Previously, directives like `ngFor` and `ngIf` were used to manipulate the DOM. However, this approach often resulted in complex and less readable code. Notably, the inability to use `ngFor` and `ngIf` together led to unnecessary DOM additions and intricate structures.

The introduction of Control Flow has enhanced code readability by separating code from HTML. Furthermore, it streamlines development by eliminating the need to import `CommonModule`. Beyond these conveniences, Control Flow offers a performance improvement of approximately 45% compared to the traditional `CommonModule` approach.

## Illustrative Examples

### Traditional `ngIf` Implementation

```ts
@Component({
	standalone: true,
	template: `
	<div *ngIf="isLoggedin">
		Welcome back, user!
	</div>`,
	imports: [
		NgIf // or CommonModule
	],
})
export class AppComponent {}

```

### Modern Control Flow Equivalent

```ts
@Component({
	standalone: true,
	template: `
	@if (isLoggedin) {
		<div>
			Welcome back, user!
		</div>
	}`,
})
export class AppComponent {}

```

## Deep Dive into `@if`

The `@if` feature conditionally renders content based on a given condition. This allows the display of different HTML blocks depending on whether a specific condition evaluates to true or false. As a result, code readability is improved, and unnecessary DOM elements can be minimized. In the example below, the first `<div>` is rendered if `condition` is true, and the second or third `<div>` is rendered if `condition` is false.

```html
{% raw %}
@if(condition) {
	<div>Content</div>
} @else if (!condition) {
	<div>Else If Content</div>
} @else {
	<div>Else Content</div>
}
{% endraw %}

```

## Harnessing the Power of `@switch`

The `@switch` feature is useful for handling multiple conditions, enabling the concise creation of complex conditional statements. It defines several cases and renders content that matches the given condition.

```html
{% raw %}
@switch(condition) {
	@case ('a') {
		<div>A</div>
	} @case ('b') {
		<div>B</div>
	} @default {
		<div>Others</div>
	}
}
{% endraw %}
```

## Mastering `@for` Loops

The `@for` feature iterates through a list, processing each item. It provides an easy way to render dynamic data. By using loops, code duplication is reduced, and flexible handling of data changes becomes possible.

```html
{% raw %}
@for(item of list; track item) {
	<li>{{item.value}}</li>
}
{% endraw %}
```

### Understanding `track`

`track` is similar to the `trackBy` feature of `ngFor`, enabling efficient tracking of each item in a list. By utilizing this feature, Angular manages DOM elements based on the unique ID of each item. `track` also detects changes in the list, reducing unnecessary DOM manipulations and improving performance. Notably, `track` is a required feature in Control Flow, making it highly important.

```html
{% raw %}
@for(item of list; track item.id) {
	<li>{{item.value}}</li>
}
{% endraw %}
```

`track` can also be extended using built-in variables.

| Built-in Variable | Description |
|---|---|
| `$index` | Row index |
| `$first` | First row |
| `$last` | Last row |
| `$even` | Even row |
| `$odd` | Odd row |

```html
{% raw %}
@for(item of list; track trackId($index, item)) {
	<li>{{item | json}}</li>
}
{% endraw %}
```
This allows setting the track through the `trackId` function.

### `track` is Mandatory When Using `for` Loops in Control Flow

If an error occurs when using `for` in Control Flow, check if `track` has been used. `track` serves the same role as `trackBy` in async pipe and is mandatory in Control Flow.

```html
@for(let item of data; track item.id) {
	<div>{% raw %}{{ item.id }}{% endraw %}</div>
}
```

### Leveraging Built-in Variables

In `@for`, you can leverage basic variables to provide additional information about each item. The example below uses built-in variables such as `$index`, `$first`, `$last`, `$even`, and `$odd`.

```html
{% raw %}
@for(item of list;let index = $index, let first = $first, let last = $last, let even = $even, let odd = $odd) {
	<li>
		{{index}}
		{{first}}
		{{last}}
		{{even}}
		{{odd}}
	</li>
}
{% endraw %}
```

## Implementing `@empty`

The `@empty` block is executed when the list is empty after performing a loop. It allows displaying a suitable message to the user when the list length is 0 or undefined.
```html
{% raw %}
@for(item of list) {
	<li>{{item | json}}</li>
} @empty {
	No Item
}
{% endraw %}
```