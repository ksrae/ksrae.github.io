---
title: "Getting All Selected Points by .getSelectedPoints()"
date: 2023-01-02 18:46:00 +0900
comments: true
categories: angular
tags: [highcharts]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Highcharts-%EC%84%A0%ED%83%9D%ED%95%9C-%EB%AA%A8%EB%93%A0-Points-%EA%B0%80%EC%A0%B8%EC%98%A4%EA%B8%B0)
<br/>

> This article explores how to retrieve information about all user-selected points in Highcharts.

## Setting the Series Option: allowPointSelect

To enable point selection via clicking, the `allowPointSelect` option must be set to `true`. This option can be configured for each series individually. Alternatively, to apply it across all series, configure it globally within the `plotOptions`.

```tsx
...
  plotOptions: {
      series: {
        allowPointSelect: true,
      }
  }
...
```

## Selecting Multiple Points

Highcharts provides Ctrl + click (on Windows) as a standard method for selecting multiple points. Unfortunately, Highcharts doesn't natively support selecting multiple points using a drag gesture. Therefore, Ctrl + click remains the primary multi-point selection method.

For those interested in implementing drag-based selection, refer to [this article](https://ksrae.github.io/angular/highcharts-select-by-draging) which explores that approach.

## Utilizing getSelectedPoints()

The `.getSelectedPoints()` method returns an array containing information about all currently selected points.

Consider a scenario where you want to retrieve this information upon a button click.

```tsx
    this.chart = Highcharts.chart({
      chart: {
        renderTo: 'chart-line',
        type: 'line',
      },
      title: { text: 'Line Chart', },
      credits: { enabled: false, },
      legend: { enabled: true, },
      yAxis: {
        title: { text: null, },
        crosshair: { snap: false, }
      },
      xAxis: {
        type: 'category',
        crosshair: { snap: false, },
        categories: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
      },
      plotOptions: {
          series: {
            allowPointSelect: true,
            borderWidth: 0,
            dataLabels: {
              enabled: true
            },
          }
      },
      series: [{
        name: 'Amount',
        data,
        linecap: 'square'
      }],

    } as any);

  onButtonClick() {
    const selectedPoints = this.chart.getSelectedPoints();

    for(let point of selectedPoints) {
      console.log(point);
    }
  }
```

Below is an example of the data structure returned.  Note that values may vary based on your specific chart configuration. Use this as a reference.

```
{
  category : {...}
  clientX: 252.29166666667
  color: "red"
  colorIndex: 0
  contrastColor: "#000000"
  dataLabel: {element: g.highcharts-label.highcharts-data-label.highcharts-data-label-color-0, onEvents: {…}, opacity: 1, renderer: C, SVG_NS: 'http://www.w3.org/2000/svg', …}
  dataLabels: [e]
  dist: 215.3084313261132
  distX: 75.04166666666998
  formatPrefix: "point"
  graphic: {element: path.highcharts-point.highcharts-point-select.highcharts-color-0, onEvents: {…}, opacity: 1, renderer: C, SVG_NS: 'http://www.w3.org/2000/svg', …}
  hasImage: undefined
  id: "highcharts-sk7rt2o-84"
  index: 3
  isInside: true
  isNull: false
  name: "1/12"
  negative: false
  options: {name: '1/12', y: 955, selected: true}
  percentage: undefined
  plotX: 252.29166666667
  plotY: 64.19200000000001
  proceed: null
  selected: true
  series: {_i: 0, chart: a, data: Array(10), eventOptions: {…}, eventsToUnbind: Array(1), …}
  shapeArgs: undefined
  state: "select"
  total: undefined
  visible: true
  x: 3
  y: 955
  yBottom: null
  zone: undefined
}
```

Crucially, this data provides not only information about the point itself, but also the series to which the point belongs and the index of the point within that series.

Leveraging this information, you can implement a variety of features, such as displaying labels at the point's location or exposing details about the point or its associated series.