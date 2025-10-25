---
title: "easy update with oneToOne Option"
date: 2022-11-28 18:11:00 +0900
comments: true
categories: angular
tags: [highcharts]
---

Highcharts offers a comprehensive set of options for data manipulation and boasts well-documented APIs, facilitating a smooth experience when using `highcharts-angular`. While the extensive documentation is valuable, locating specific commands for every scenario can be cumbersome. Often, the most straightforward and efficient approach is to directly inject values for immediate application.

This post will explore a method for configuring charts by immediately applying changes to the `options` object.

## Conventional Chart Implementation

Typically, charts are implemented as follows:

```html
<highcharts-chart 
  [Highcharts]="Highcharts"
  [options]="chartOptions">
</highcharts-chart>
```

```tsx
export class HighchartsComponent implements OnInit {
  Highcharts = Highcharts;
  chartOptions: Highcharts.Options;

   ngOnInit(): void {

    this.chartOptions = {
      chart: {
        renderTo: 'chart-line',
        type: 'line',
      },
      series: [{
        data: [7.0, 6.9, 9.5, 14.5, 18.4, 21.5, 25.2, 26.5, 23.3, 18.3, 13.9, 9.6]
      }],
    }
  }
}
```

To add a new series to this chart, one would typically consult the API documentation to identify the appropriate function and apply it as shown below:

```tsx
this.Highcharts.addSeries({
  data: [1,2,3,4,5,6,7,8,9,0]
});
```

This approach requires familiarity with the entire API, which can be inconvenient. To mitigate this, we will explore a method for directly modifying the `options` object.

## Enhanced Chart Implementation

Remarkably, direct modifications to the `chartOptions` object are reflected in the chart. For instance, to change the chart type to a bar chart, the following modification suffices:

```tsx
this.chartOptions = {
  ...this.chartOptions,
  chart: {
    ...this.chartOptions.chart,
    type: 'bar'
  }
}
```

However, changes to the `series` property, as demonstrated above, will not be applied by default. This behavior stems from the `onetoOne` option's default value being set to `false`. (Refer to the [previous article](/highcharts-angular) for more details).

However, by setting `onetoOne` to `true` and modifying the `series`, the changes are applied.

```html
<highcharts-chart 
  [Highcharts]="Highcharts"
  [options]="chartOptions"
  [onetoOne]="true">
</highcharts-chart>
```

```tsx

...
const newSeries = {
  data: [1,2,3,4,5,6,7,8,9]
};

this.chartOptions = {
  ...this.chartOptions,
  series: [...this.chartOptions.series, newSeries]
}

```

The primary advantage of this approach lies in bypassing the need to consult the API for every modification. This allows for quick and easy modification of various chart options, streamlining the development process.