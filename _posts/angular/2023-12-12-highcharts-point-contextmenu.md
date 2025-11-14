---
title: "Setting Contextmenu Event to all Points in a series of Highcharts without  highcharts-custom-events"
date: 2023-12-12 14:08:00 +0900
comments: true
categories: angular
tags: [highcharts, contextmenu]
---


Highcharts does not natively provide a `contextmenu` event for data points. While installing the `highcharts-custom-events` package from npm is a common solution, this blog post explores how to implement context menu events on Highcharts data points without relying on third-party libraries.

## Solution

### 1. Registering the Event Handler During Chart Creation

Leverage the `load` event when creating the Highcharts chart to register a context menu event handler for every data point in each series.  This involves utilizing a method from your Angular component to handle the context menu events.

```tsx
events: {
  load: (e: any) => {
    for(let series of e.target.series) {
      for(let point of series.points) {
        point.graphic.on('contextmenu', (f: any) => {
          f.preventDefault();
          console.log('contextmenu', {point});
          return false;
        });
      }
    }
  }
},
```

### 2. Implementing the Context Menu

When implementing the context menu, use `preventDefault` to prevent the browser's default context menu from appearing. Define your custom actions within the event handler.  The example above simply logs to the console, but in a real-world project, you would add more complex logic, such as displaying a custom menu or triggering specific data operations.

### 3. Enabling Point Selection

To receive point events, ensure that `allowPointSelect` is set to `true` for all series.  Using `plotOptions` is the most efficient way to apply this setting globally.

```tsx
plotOptions: {
  series: {
    allowPointSelect: true,
  }
}
```

## Conclusion

This approach enables Highcharts charts to provide custom context menus for each data point without requiring additional libraries. This enhances the user experience by providing easy access to additional information or actions related to the visualized data, improving overall interactivity and usability.


## Example Codes
```ts
import { ChangeDetectionStrategy, Component, OnInit } from '@angular/core';
import * as Highcharts from 'highcharts';

@Component({
  selector: 'app-contextmenu',
  standalone: true, // standalone: true 추가
  template: `<div id="chart-container"></div>`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ContextMenuComponent implements OnInit {
  chart: Highcharts.Chart | undefined;

  ngOnInit(): void {
    this.createChartLine();
  }

  private createChartLine(): void {
    this.chart = Highcharts.chart('chart-container', {
      chart: {
        type: 'line',
        events: {
          load: (e: Highcharts.ChartLoadEventObject) => {
            const chart = e.target as Highcharts.Chart;
            for (const series of chart.series) {
              for (const point of series.points) {
                // point.graphic?.element.addEventListener('contextmenu', ...) 와 동일
                point.graphic?.on('contextmenu', (event: Event) => {
                  event.preventDefault();
                  console.log('Context menu on point:', { point });
                  // 여기에 커스텀 메뉴 로직을 구현합니다.
                  return false;
                });
              }
            }
          }
        },
      },
      title: {
        text: 'Context Menu on Points Example'
      },
      xAxis: {
        categories: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
      },
      series: [{
        type: 'line', // type 명시
        data: [29.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5, 216.4, 194.1, 95.6, 54.4],
      }],
      plotOptions: {
        series: {
          allowPointSelect: true,
          // point.events.contextmenu는 기본적으로 지원되지 않음
        },
      },
    });
  }
}

```