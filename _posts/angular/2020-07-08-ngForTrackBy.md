---
title: "ngFor with TrackBy"
date: 2020-07-08 14:25:00 +0900
comments: true
categories: angular
tags: [ngfor, trackby]
---


`ngFor` is a structural directive in Angular that iterates over a collection and renders a template for each item. While powerful, it has a potential performance drawback: by default, any change within the loop triggers a refresh of *all* DOM elements rendered by `ngFor`. This can lead to unnecessary re-renders and performance degradation, especially with large datasets.

```html
<div *ngFor="let item of items;let idx=index;"></div>
<!-- or -->
<div *ngFor="let item of items;index as idx;"></div>
```

## Optimizing `ngFor` with `trackBy`

The `trackBy` option provides a mechanism to optimize `ngFor`'s rendering behavior. It allows Angular to identify which items in the collection have actually changed, and only update the corresponding DOM elements. This significantly reduces the number of re-renders, resulting in improved performance.

To use `trackBy`, you need to provide a function that uniquely identifies each item in the collection. This function should return a stable identifier, typically a primary key or unique ID.

```tsx
let items = [
	{ id: 1, name: 'first'},
	{ id: 2, name: 'second'},
	{ id: 3, name: 'third'},
	{ id: 4, name: 'fourth'}
];

trackByItem = (item: any): number => item.id;
```

In this example, the `trackByItem` function returns the `id` property of each item, which is assumed to be unique. Now, you can use this function in your template:

```html
<div *ngFor="let item of items;trackBy: trackByItem"></div>
```

With `trackBy` in place, Angular will only re-render DOM elements corresponding to items whose identifiers have changed.

## Angular Version Considerations

In some Angular versions, the `trackByItem` function signature might require the `index` as the first parameter. If you encounter errors related to the function's arguments, adjust the signature to include the index:

```tsx
trackByItem = (index: number, item: any): number => item.id;
```

This ensures compatibility with newer Angular versions.


## Angular 17+ - Control flow

There are two primary ways to use track with @for<br/>
<br/>
1. Tracking by a Unique Property (Recommended & Simplest)
The most common and readable method is to track an item directly by its unique property. You no longer need a separate function in your component for this simple case.

```html
<!-- New way with @for block -->
@for (item of items; track item.id) {
  <div>{{ item.name }}</div>
}
```
Here, track item.id tells Angular to use the id property of each item as its unique identifier.

2. Tracking with a Function
If your tracking logic is more complex (e.g., combining multiple properties), you can still use a function. The syntax is slightly different from *ngFor.

```html
<!-- Using the same trackByItem function from before -->
@for (item of items; track trackByItem) {
  <div>{{ item.name }}</div>
}
```


## Additional Resources

- [Angular `ngFor` Documentation](https://angular.io/guide/template-syntax#ngfor-with-trackby)
- [Stack Overflow Discussion on `trackBy`](https://stackoverflow.com/questions/42108217/how-to-use-trackby-with-ngfor)