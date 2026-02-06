---
title: "All about Highcharts Tooltip (Options, Clone, Customize)"
date: 2023-09-26 13:49:00 +0900
comments: true
categories: angular
tags: [highcharts]
---

## Tooltip Configuration in Highcharts

Let's delve into the tooltip options available in Highcharts. Tooltips enhance user interaction by displaying contextual information when hovering over data points. Highcharts provides a range of options to customize tooltips for optimal data presentation.

```tsx
  tooltip: {
    headerFormat: '<table><tr><th colspan="2">{point.key}</th></tr>', // Displays at the top of the tooltip; boldness is automatically applied.  The 'point' and 'series' values can be used directly without separate definitions. Any value from the series can also be retrieved.
    pointFormat: `<tr><td style="color: {series.color}">{series.name} {point.options.custom.test} {series.options.custom.test}</td>` +
        '<td style="text-align: right"><b>{point.y} EUR</b></td></tr>', // Displayed with the line color. Typically used to show the primary content.
    footerFormat: '</table>', // Displays at the bottom of the tooltip.
    valueDecimals: 2, // Number of decimal places to display.
    shared: false, // Displays all information for the same x-coordinate in a single tooltip. The format is also unified.
    split: false, // Displays tooltips separately.
    useHTML: true, // Determines whether HTML can be used in the format. If false, HTML cannot be applied in the format.
    followPointer: true, // The tooltip follows the mouse cursor.
    // positioner: () => { return {x: 80, y: 50}}, // Fixes the tooltip position.
    shadow: false, // Determines whether to display a shadow.
    // crosshairs: true, // deprecated
  },
  series: [{
  name: 'Amount',
  data,
  linecap: 'square',
  custom: {
    test: 'test data',
  }
}]

```

**Explanation:** The `tooltip` object within the Highcharts configuration allows granular control over the tooltip's appearance and content. Key properties include `headerFormat`, `pointFormat`, and `footerFormat` for structuring the tooltip's sections.  The `useHTML` property is particularly important as it enables the use of HTML tags within the format strings for richer styling.

## Duplicating Tooltips

An advanced technique involves duplicating tooltips to display multiple instances. This is useful for dynamically adding tooltip-like elements to the chart. It's crucial to note that when the `useHTML` option is set to `true`, only the text content is cloned. Therefore, this approach is best suited for tooltips that do not heavily rely on HTML formatting.

```tsx
 this.chart = Highcharts.chart({
  ...
  plotOptions: {
    series: {
      point: {
        events: {
          click: (p: any) => {
            let cloneTooltip = this.chart.series[0].chart.tooltip.label.element.cloneNode(true);
            this.chart.container.firstChild.appendChild(cloneTooltip);
          },
        },
      }
    }
  },
  tooltip: {
    shared: false,
    enabled: true
  },
```

**Implementation Notes:** The code snippet demonstrates how to clone the existing tooltip element and append it to the chart's container. This is achieved by accessing the `tooltip.label.element` property and using the `cloneNode(true)` method.

## Custom Tooltip Implementation

Another customization strategy involves disabling the default Highcharts tooltip and implementing a custom label that mimics tooltip behavior. This provides maximum flexibility in designing the tooltip's appearance and functionality.<br/>

To implement this method, you need to configure the following options in the default tooltip options:

- `enabled: false`
- `followPointer: false`

```tsx
  this.chart = Highcharts.chart({
    ...
    tooltip: {
      enabled: false, // disable the default one
      followPointer: false
    },
    plotOptions: {
      series: {
        point: {
          events: {
            click: (p: any) => {
              this.chart.renderer.label(
                'tooltip title', // title
                this.chart.plotLeft + p.point.plotX, // x position
                this.chart.plotTop + p.point.plotY // y position
              )
              .attr({
                  padding: 10, // this is an example, change to any number
                  r: 5, // this is an example, change to any number
                  fill: '#000',
                  zIndex: 5 // this is an example, change to any number
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

**Code Breakdown:** The provided code demonstrates how to create a custom label using the `chart.renderer.label()` method.  The label's position is dynamically calculated based on the clicked data point's coordinates (`p.point.plotX`, `p.point.plotY`).  Attributes such as padding, border radius (`r`), background color (`fill`), and `z-index` are then applied to style the custom tooltip. CSS styles, such as text color, can also be applied.<br/><br/>

In conclusion, this exploration of Highcharts tooltip options and customization techniques empowers you to enhance data visualization and improve the overall user experience.<br/><br/>

**Reference:** The examples provided are adapted from the highcharts jsfiddle code, with modifications to the typescript version.<br/>

For further inquiries or assistance with code modifications, please feel free to leave a comment or send an email.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Highcharts-tooltip-%EC%BB%A4%EC%8A%A4%ED%85%80%ED%95%98%EA%B8%B0)
<br/>