---
title: "How to Use This of Highchart in Typescript"
date: 2022-10-21 13:49:00 +0900
comments: true
categories: angular
tags: [highcharts]
---

Searching for examples, one can often find examples in JavaScript or jQuery that use `this`. However, `this` has a different meaning in TypeScript or Angular, so simply copying the code can cause errors. This article explores how to resolve such errors.

## Understanding `this` in JavaScript

A quick search on Google reveals many questions about the meaning of `this` in Highcharts.

For example, you might encounter the following scenario:

```jsx
tooltip: {
  formatter: function() {
    return '<b>'+ (this.point ? this.point.name : this.series.name)  +'</b><br/>';
  }
},
```

How can this be applied in TypeScript or Angular? In the code above, the formatter is defined as a function without parameters, but in TypeScript or Angular, it can accept parameters. This parameter corresponds to the `this` value above. Therefore, the code can be modified and applied as follows:

```tsx
formatter: (e: any) => {
  return '<b>'+ (e.point ? e.point.name : e.series.name)  +'</b><br/>';
}
```

In the example above, the `any` type is used for convenience, but referring to the Highcharts API documentation allows you to identify the precise type. For instance, the type of `this` in the formatter is `Highcharts.TooltipFormatterContextObject`, so you can modify the code as follows (see [link](https://api.highcharts.com/class-reference/Highcharts#.TooltipFormatterCallbackFunction)):

```tsx
formatter: (e: Highcharts.TooltipFormatterContextObject) => {
  return '<b>'+ (e.point ? e.point.name : e.series.name)  +'</b><br/>';
}
```

## Reference

[Highcharts-JS API Reference](https://api.highcharts.com/highcharts/)