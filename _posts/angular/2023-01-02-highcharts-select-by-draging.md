---
title: "Multiple Point Selection by Draging"
date: 2023-01-02 18:46:00 +0900
comments: true
categories: angular
tags: [highcharts]
---

> This article explores how to implement point selection via mouse drag in Highcharts, a feature not natively supported by the library.

## Implementing Drag-to-Zoom for Point Selection

Highcharts does not provide built-in functionality for dragging the chart background. To achieve this behavior, we need to repurpose existing features. The `Zoom` functionality combined with the `selection event` offers the most suitable approach for implementing multi-selection.

## Configuring Zoom Settings

First, set the `zoomType` to 'xy'. This enables drag functionality across both the x and y axes. Additionally, define the `selection` event within the chart's event configuration.

Here's a code example illustrating this setup:

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

## Handling the Selection Event

Now, let's delve into the implementation of the `selectPointsByDrag` function. The logic is as follows:

- The `selection` event triggers upon the completion of the drag operation.
- The event's parameter contains the coordinates of the selected area.
- We iterate through the data of each series, filtering points that fall within the coordinates specified by the parameter.

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
  *  Insert code to handle the selection result here.
  */

  return false;
}
```

## The Significance of `return false`

Removing the `return false;` line will cause an immediate zoom action after the selection is complete. The `return false;` statement effectively prevents the default zoom behavior, which is crucial for isolating the selection functionality.

## Conclusion

Highcharts provides a solid foundation for charting. While its core functionalities are reliable, it may lack certain features and present challenges for extensive customization.

However, the active community has contributed numerous implementations and extensions. Leveraging these resources can enable the creation of highly customized and powerful charts using Highcharts.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Highcharts-Drag%EB%A1%9C-%EB%8B%A4%EC%A4%91-%ED%8F%AC%EC%9D%B8%ED%8A%B8-%EC%84%A0%ED%83%9D%ED%95%98%EA%B8%B0)
<br/>