---
title: "Highcharts Drag로 다중 포인트 선택하기 - Multiple Point Selection by Draging"
date: 2023-01-02 18:46:00 +0900
comments: true
categories: angular
tags: [highcharts]
---

> 이 글에서는 Highchart에서 지원하지 않는 마우스 drag로 point를 선택하는 방법을 알아봅시다.
<br/>

## How to Drag - Zoom
Highchart의 기본 기능에는 chart의 background를 drag 하는 기능이 없습니다. <br/>
이를 구현하려면 drag를 지원하는 다른 기능을 변형하여 사용하여야 하는데<br/>
`Zoom` 의 기능과 `selection event` 가 우리가 원하는 multi selection을 기능을 구현하는데 가장 적합합니다.<br/>
<br/>

## Zoom 설정

우선 ZoomType을 'xy'로 설정합니다. 이를 통해 x,y 축 영역을 다 활용하여 drag 를 적용할 수 있습니다.<br/>
또한 chart event의 selection 이벤트를 정의 합니다.<br/>
<br/>
구현하면 다음과 같습니다.<br/>

```tsx
  this.chart = Highcharts.chart({
    chart: {
      renderTo: 'chart-line',
      type: 'line',
    },
    events: {
      selection: (e: any) => this.selectPointsByDrag(e)
    },
    zoomType: 'xy',
    series: [
      { data: [10, 5, 0, -5, -10, -15, -10, -10, -5, 0, 5] },
    ]
  });
```
<br/>

## Selection Event
이제 함수를 구현해 봅시다. 구현 방식은 다음과 같습니다.<br/>
- selection 이벤트는 drag가 끝나는 시점에 실행됩니다.
- selection 이벤트의 파라미터 값은 선택한 영역의 좌표를 가지고 있습니다.
- 따라서 모든 series의 데이터를 검사하여 파라미터의 좌표 내에 있는 항목만 걸러냅니다.

```tsx
selecPointsByDrag(e: any) {
  let group: any[] = [];

  for(let series of e.target.series) {
    let list = [];
    for(let point of series.points) {
      if (point.x >= e.xAxis[0].min && point.x <= e.xAxis[0].max &&
          point.y >= e.yAxis[0].min && point.y <= e.yAxis[0].max) {

            // console.log(point.options)
        list.push([point.options?.name ? point.options.name : point.options.x, point.options.y]);
      }

    }
    group.push(list);
  }

  /* 
  * 여기에 select 한 이후에 처리할 코드가 들어가야 함
  */

  return false;
}
```
<br/>

## Return false
만일 `return false;` 항목을 제거하고 실행하면 selection이 실행되자마자 곧바로 zoom이 실행되는 것을 볼 수 있습니다.<br/>
즉, `return false;` 는 zoom 기능을 동작하지 않도록 막아주는 중요한 역할을 합니다.<br/>
<br/>

## 결론
Highchart는 기본에 충실한 차트입니다. 이 말은 차트가 지원하는 기본 기능은 매우 안정적으로 동작한다는 의미이기도 하지만, 많은 기능이 구현되어 있지 않고 커스텀하기에 쉽지 않다는 의미이기도 합니다.<br/>
<br/>
하지만, 여러 커뮤니티를 통해 많은 개발자들이 구현한 내용이 있으므로 이를 잘 활용하면 확장된 좋은 차트를 구현할 수 있을 것입니다.<br/>



