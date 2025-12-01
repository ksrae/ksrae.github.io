---
title: "What is charts variables of Highcharts in Highcharts-angular?"
date: 2022-11-15 13:49:00 +0900
comments: true
categories: angular
tags: [highcharts]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Highcharts-%EC%97%90%EC%84%9C-chart-%EB%B3%80%EC%88%98-%EC%A0%91%EA%B7%BC%ED%95%98%EA%B8%B0)
<br/>

In Highcharts, you can directly inject options into the `chart` variable, making it easy to utilize the API and modify values. However, in highcharts-angular, attempting to apply options directly to the `chart` variable leads to a different outcome.

This article explores how to access the `charts` variable in highcharts-angular, similar to how it's done in Highcharts.

## Accessing the Chart Variable in Highcharts

In Highcharts, you directly inject `Highcharts.chart` into the `chart` variable, as shown below:

```tsx
export HighchartsComponent implements OnInit {
  chart: any;

  ngOnInit() {
    this.chart = Highcharts.chart({
      ...
    })
  }
}
```

Therefore, accessing the `chart` variable directly is straightforward.

```tsx
const series = this.chart.series;
```

## Accessing the Chart Variable in Highcharts-angular

In Highcharts-angular, accessing options from the `chart` variable appears complex because the `chart` variable's structure differs from that of Highcharts. Additionally, directly accessing the visible `Chart` variable is restricted and inconvenient.

## Solution 1 - Highcharts.charts[0]

The solution is to access the `charts[]` variable, which is an array, instead of the directly visible `Chart` variable. If there's only one chart, you can access it via `charts[0]`. This will give you a value equivalent to the `chart` variable in Highcharts.

In essence, `this.chart` in Highcharts is equivalent to `this.Highcharts.charts[0]` in highcharts-angular.

```tsx
const series = this.Highcharts.chart[0].series;
```

## Solution 2 - callbackFunction

The disadvantage of the previous method is that you need to know the chart's index. If the index is incorrect, you might inadvertently issue commands to the wrong chart. In contrast, `callbackFunction` is easier to use because the parameter value *is* the chart.

The `callbackFunction` provides its own `chart` variable during chart creation. Since this variable always represents the chart being controlled, there's less room for error, and because the value is received at chart creation, there's less concern about commands being injected before creation.

The implementation is as follows. This example demonstrates creating code that receives the callback and passes it up.

```html
<highcharts-chart>
  [Highcharts]="Highcharts"
  [options]="highchartsOptions"
  ...
  [callbackFunction]="chartCallback"
</highcharts-chart>
```

```tsx
...
export class HighchartsComponent {
  ...
  chartCallback: Highcharts.ChartCallbackFunction = (chart: Highcharts.Chart) => {
    this.chartInstance.emit(chart);
  }
}
```

**Caution:**

The biggest issue with this method is that print or export image functionalities using the chart command can only be executed once. Executing them multiple times results in a "chart value is undefined" error, and the functions will not work.

## Solution 3 - chartInstance

highchart-angular creates a `chartInstance` upon creation. This method uses output to pass this value externally for use.
This method provides the same functionality as the callback above, but it also resolves the issue discovered in the callback.
In other words, print and export functionalities can be executed multiple times without issue.

```html
<highcharts-chart>
  [Highcharts]="Highcharts"
  [options]="highchartsOptions"
  ...
  (charts)="chartInstance($event)"
</highcharts-chart>
```

```tsx
...
export class HighchartsComponent {
  ...
  @Output() chartInstance = new EventEmitter<Highcharts.Chart>();
}
```

```tsx
@Component: {
  template: '<app-highcharts (chartInstance)="setChartInstance($event)"></app-highcharts>
  ...
}
export class HighchartsContainerComponent {
  setChartInstance(e: any) {

  }

}
```