---
title: "All about Highcharts Tooltip (Options, Clone, Customize)"
date: 2022-10-21 13:49:00 +0900
comments: true
categories: angular
tags: [highcharts, highcharts-angular]
---

> 이 글에서는 Highchart의 Tooltip에서 사용가능한 Options 설명과 Clone, 그리고 Customizing 방법에 대해서 알아보겠습니다.
 
## Tooltip Options
```tsx
  tooltip: {
    headerFormat: '<table><tr><th colspan="2">{point.key}</th></tr>', // 툴팁 상단 표시, 굵기가 자동 설정됨. point와 series 값을 별도의 정의 없이 사용할 수 있음. series의 임의의 값도 가져올 수 있음.
    pointFormat: `<tr><td style="color: {series.color}">{series.name} {point.options.custom.test} {series.options.custom.test}</td>` +
        '<td style="text-align: right"><b>{point.y} EUR</b></td></tr>', // 라인의 색으로 표시됨. 일반적으로 내용을 표시할 때 사용.
    footerFormat: '</table>', // 툴팁 하단 표시.
    valueDecimals: 2, // 소수점 표시 자릿수.
    shared: false, // 동일한 x 좌표의 모든 정보를 하나의 툴팁에 표현. 포멧도 통일됨.
    split: false, // 툴팁을 여러 개로 표시
    useHTML: true, // Format에서 html 사용 여부. false일 때는 Format에서 html을 적용할 수 없음.
    followPointer: true, // 마우스가 어디든 툴팁이 따라옴
    // positioner: () => { return {x: 80, y: 50}}, // 위치 고정
    shadow: false, // 그림자 표현 여부
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

## Tooltip Clone

Tooltip을 복사하여 여러 개를 띄우는 방법 입니다.
단 이 방법은 useHTML이 true인 경우에는 text가 없는 껍데기만 복사되므로 이 방법을 활용하려면 <b>Tooltip에 html 이 없어야 합니다.</b>

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


## Tooltip Customized

기존 tooltip clone 방법은 html을 사용할 수 없다는 큰 단점이 있으므로 이를 극복하기 위한 다른 방법을 알아봅시다. 
이 방법은 lable을 Tooltip 처럼 표시하는 방법입니다. point click을 통해 Tooltip과 동일한 동작을 구현하는 방법을 적용하였습니다.

이 방법을 적용하려면 기본 tooltip 옵션 중 몇가지 옵션을 다음과 같이 설정해야 합니다.
- enabled; false
- followPointer: false


```tsx
  this.chart = Highcharts.chart({
    ...
    tooltip: {
      enabled: false // disable the default one
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


위의 예제들은 highcharts의 jsfiddle에 있는 코드를 참조하여, 제가 직접 typescript 버전으로 수정한 코드입니다. 
만일 highcharts-angular에 적용이 필요한 코드나 수정한 코드에 대한 문의는 댓글이나 이메일 주시면 답변 드리겠습니다.
