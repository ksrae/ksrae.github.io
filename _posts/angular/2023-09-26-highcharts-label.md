---
title: "Highcharts Label - Changing Attributes Dynamically)"
date: 2023-09-26 13:49:00 +0900
comments: true
categories: angular
tags: [highcharts]
---

In this article, we'll explore how to modify the style of labels in Highcharts.

## Analysis

Modifying the attributes of a label directly can be challenging.<br/>

To understand why, examining the HTML structure of a label reveals that it's a combination of a `<g>` tag and a separate `<div>`.<br/>

When attempting to set up label events in a component and retrieve the DOM element displaying the label, the `<g>` tag is typically captured. The issue is that the desired attribute settings are associated with the `<div>` tag.<br/>

To address this, we'll employ some workarounds.<br/>

Let's start by extending a customized tooltip example from a previous discussion. The original customized tooltip looked like this:

```tsx
  this.chart = Highcharts.chart({
    ...
    tooltip: {
      enabled: false // disable the default one
    },
    plotOptions: {
      series: {
        point: {
          events: {
            click: (p: any) => {
              this.chart.renderer.label(
                'tooltip title', // title
                p.offsetX, // x position
                p.offsetY, // y position
              )
              .attr({
                  padding: 10,
                  r: 5,
                  fill: '#000',
                  zIndex: 5
              })
              .css({
                  color: 'white'
              })
              .add();
            },
          },
        }
      }
    },
  });
```

## Setting Attributes on Labels at Initialization

To apply styles to a label, it needs to accept HTML-formatted data. Therefore, the text needs to be converted to HTML.

### Utilizing `useHTML`

It's also necessary to enable HTML usage. This option is provided in the `renderer.label` options.

```
SVGRenderer.label(str: string, x: number, y?: number, shape?: string, anchorX?: number, anchorY?: number,
useHTML?: boolean, baseline?: boolean, className?: string): Highcharts.SVGElement
```

To apply a class to the entire label, set the `className` option. To apply classes or styles to individual elements within the label, use `useHTML`. We'll utilize `useHTML` here.

The example above can be modified as follows:

```tsx
  ...
  this.chart.renderer.label(
    `<p>Title</p><button type="button" class="btn">Click to Confirm<button>`,
    p.offsetX, 
    p.offsetY, 
    null, null, null, true
  )
  ...
```

### Storing the Label in a Variable

The configured label must be stored in a variable.<br/>

The reason for storing it in a variable is for DOM access, which we'll discuss further later.

```tsx
let label =   
  this.chart.renderer.label(
    `<p>Title</p><button type="button" class="btn">Click to Confirm<button>`,
    p.offsetX, 
    p.offsetY, 
    null, null, null, true
  );
```

### Defining Events

If an event is required for the label, it can be defined using the `on` function.<br/>

For instance, if we want to change the class of the label when it's clicked, we need to define a click event.<br/>

Let's create code that adds a class name when the label is clicked.<br/>

The click event can be set as follows:

```tsx
label.on('click', (event: any) => {

});
```

It's important to note that the DOM targeted by the event is the `<g>` tag. The value we'll use is the `element` property of the `label` variable. This belongs to the `<div>` and can be used to change the class. Access to buttons or other DOM elements within the `<div>` is also possible.

```tsx
label.on('click', (event: any) => {
  label.element.addClass('active'); // example of setting class to label
  label.element['div'].children[0].childNodes[0].style.color = 'red'; // example of setting firstNode style
});
```