---
title: "How to setup highcharts-angular"
date: 2022-10-21 13:49:00 +0900
comments: true
categories: angular
tags: [chart, highcharts]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Highcharts%EB%A5%BC-Angular%EC%97%90%EC%84%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
<br/>

Highcharts-Angular is a library designed to simplify the integration of Highcharts into Angular applications.

This guide will cover the installation of Highcharts and Highcharts-Angular, followed by an overview of the essential input properties provided by Highcharts-Angular.

## Rationale for Using Highcharts-Angular

While Highcharts can be utilized in Angular without additional libraries, as demonstrated below, Highcharts-Angular offers significant advantages.

For instance, you can directly render a Highcharts chart in Angular like this:

```html
<div id="chart-line"></div>
```

```tsx
import * as Highcharts from 'highcharts';
@Component({
  ...
})
export class HighchartsComponent implements OnInit {
  chart: any;

  ngOnInit(): void {
    this.chart = Highcharts.chart({
      chart: {
        renderTo: 'chart-line',
      },
      series: [{
        data: [1, 2, 3],
        type: 'line'
      }],
    } as any)
  }
}
```

### Benefits of Highcharts-Angular

Consider using Highcharts-Angular for the following reasons:

- **Zone.js Integration:** Simplifies handling zone.js events. Avoid manual event triggering or executing logic outside Angular zones.
- **Simplified Component Structure:** Reduces code complexity by injecting options directly into the component, eliminating the need to create Highcharts instances via `Highcharts.chart` and manage them manually. Implementing the equivalent functionality with vanilla Highcharts requires an observable pattern, adding unnecessary complexity.
- **Chart Instance Access:** Provides the option to output and utilize the chart instance from Highcharts-Angular, offering flexibility without sacrificing code simplicity.
- **Declarative Configuration:** Enables chart customization through options, minimizing the need for direct manipulation of chart commands. (A detailed discussion on this will follow in a future post.)

## Setup Instructions
To begin, ensure that Highcharts and the Highcharts-Angular wrapper are installed.

```bash
npm i highcharts
npm i highcharts-angular
```

In modern Angular applications using standalone components, you no longer need to declare HighchartsChartModule in an NgModule. Instead, you import it directly into the component that will use the chart.
Import HighchartsChartModule into the imports array of your standalone component's decorator like this:

```tsx
import { Component } from '@angular/core';
import { HighchartsChartModule } from 'highcharts-angular';

@Component({
  selector: 'app-your-chart-component',
  standalone: true,
  imports: [
    HighchartsChartModule // Import the module here
  ],
  template: `<!-- Your highcharts-chart component will go here -->`
})
export class YourChartComponent {
  // Component logic
}
```

## Usage Guide

### Template Configuration

Highcharts-Angular provides the following essential options:

```html
<highcharts-chart 
  [Highcharts]="Highcharts"
  [constructorType]="chartConstructor"
  [options]="chartOptions"
  [callbackFunction]="chartCallback"
  [(update)]="updateFlag"
  [oneToOne]="oneToOneFlag"
  [runOutsideAngular]="runOutsideAngularFlag">
</highcharts-chart>
```

Attribute Details:

- `highcharts: typeof Highcharts // required`: Pass the Highcharts variable.
- `chartConstructor: string = 'chart'; // optional`: Defines the chart type. Available options include 'chart', 'stockChart', 'mapChart', 'ganttChart'.
- `chartOptions: Highcharts.Options = { ... }; // required`: Specifies chart configuration values.
- `chartCallback: Highcharts.ChartCallbackFunction = function (chart) { ... } // optional`: Callback function executed after chart creation.
- `updateFlag: boolean = false; // optional`: Forces an update after chart creation. Set to true to trigger an update; it automatically resets to false after the update.
- `oneToOneFlag: boolean = true; // optional`: Controls the addition or removal of series, xAxis, and yAxis. Unlike `updateFlag`, this appends or removes values instead of replacing existing ones.
- `runOutsideAngular: boolean = false; // optional`: Prevents change detection. Use in conjunction with `updateFlag` for improved performance.

### Component Implementation

Let's examine how to define the required properties mentioned above and import Highcharts. Note that Highcharts does not provide a dedicated d.ts file; therefore, all declared values must be imported.

```tsx
import * as Highcharts from 'highcharts';

...
export class HighchartsComponent {
  Highcharts = Highcharts;
  chartConstructor = 'chart';
  chartOptions = {
    series: [{
      data: [1, 2, 3],
      type: 'line'
    }]
  };
  chartCallback;
  updateFlag = false;
  oneToOneFlag = false;
  runOutsideAngularFlag = false;
}
```

## Module/Plugin Loading

Loading additional Highcharts modules in TypeScript/Angular requires a specific approach: Import the module and then inject the Highcharts variable defined in the imported module.

The same applies to plugins.

```tsx
import HC_exporting from 'highcharts/modules/exporting';
HC_exporting(Highcharts);
```

## Server-Side Rendering (SSR)

When using SSR, prevent errors by ensuring that the chart variable is defined before rendering the component. Use `ngIf` to conditionally render the `highcharts-chart` component.

Otherwise, Highcharts-Angular might attempt to use an undefined variable.

```html
<highcharts-chart 
  *ngIf="Highcharts"
  [Highcharts]="Highcharts"
  [constructorType]="chartConstructor"
  [options]="chartOptions"
  [callbackFunction]="chartCallback"
  [(update)]="updateFlag"
  [oneToOne]="oneToOneFlag"
  [runOutsideAngular]="runOutsideAngularFlag">
</highcharts-chart>
```

## Reference
[Highcharts-Angular](https://github.com/highcharts/highcharts-angular)