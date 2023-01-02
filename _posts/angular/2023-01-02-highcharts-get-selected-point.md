---
title: "Highcharts 모든 선택한 Points 가져오기 - Getting All Selected Points by .getSelectedPoints()"
date: 2023-01-02 18:46:00 +0900
comments: true
categories: angular
tags: [highcharts, highcharts-angular]
---

> 이 글에서는 Highchart에서 유저가 선택한 모든 point의 정보를 가져오는 방법을 알아봅시다.



## Set Series Option - allowPointSelect
각 point를 클릭하려면 반드시 allowPointSelect 옵션이 true로 설정되어야 합니다.<br/>
이 옵션은 각 series에서 설정할 수도 있으나 모든 series에서 설정하려면 간단히 plotOpions에서 설정할 수도 있습니다.<br/>

```tsx
...
  plotOptions: {
      series: {
        allowPointSelect: true,
      }
  }
...
```
<br/>

## How to Select Multi Points
Highchart 에서는 유저가 여러 개의 포인트를 선택하는 방법으로 Ctrl + click (Windows) 을 제공합니다.<br/>
아쉽게도 Drag로 여러 개의 포인트를 선택하는 방법은 일반적으로 Highchart가 제공하지 않으므로 Ctrl + click (Windows) 이 유일한 Multi Points Selection의 방법입니다.<br/>
<br/>
Drag를 활용하는 방법은 [다음 글](https://ksrae.github.io/angular/highcharts-select-by-draging)에서 알아봅시다.<br/>
<br/>

## getSelectedPoints()
.getSelectedPoints() 는 선택한 여러 개의 포인트의 정보를 제공해줍니다.<br/>
<br/>
만일 어떠한 버튼을 클릭했을 때 정보를 가져온다고 합시다.<br/>

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

아래는 제가 테스트 한 결과입니다. 설정에 따라 값들은 달라질 수 있으므로 참고만 하시기 바랍니다.<br/>

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
<br/>
중요한 포인트는 이 값을 통해 단순히 point의 정보만 알 수 있는 것이 아니라 이 point가 포함된 series의 값을 알 수 있다는 점과
series에서 몇 번째 포인트를 클릭했는지 정보를 알 수 있다는 점 입니다.<br/>
<br/>
이를 활용하여 해당 포인트의 위치에 label을 보여주거나 또는 해당 point 또는 point가 속한 series의 정보를 노출하는 등 다양한 방법으로 활용할 수 있습니다.<br/>
<br/>
