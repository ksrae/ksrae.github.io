---
title: "All about Highcharts Tooltip (Options, Clone, Customize)"
date: 2023-09-26 13:49:00 +0900
comments: true
categories: angular
tags: [highcharts]
---



## Tooltip Options
먼저, Highcharts의 툴팁 옵션에 대해 알아봅시다. 툴팁은 데이터 포인트를 가리킬 때 정보를 표시하는데 사용됩니다. 다양한 옵션을 통해 툴팁을 사용자 정의할 수 있습니다.<br/>
<br/>


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

툴팁을 복사하여 여러 개를 표시하는 방법도 있습니다. 이렇게 하면 툴팁을 동적으로 추가할 수 있습니다. 하지만 useHTML 옵션이 true로 설정된 경우, 텍스트만 복사되므로 HTML이 없는 툴팁에서 사용해야 합니다.<br/>
<br/>

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


## Tooltip Customizing
툴팁을 커스터마이징하는 또 다른 방법은 기본 툴팁을 비활성화하고, 사용자 정의 레이블을 툴팁처럼 표시하는 것입니다. 이를 통해 사용자가 원하는 형태로 툴팁을 디자인할 수 있습니다.<br/>
<br/>
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
위의 코드 예제를 통해 툴팁 옵션 설정 및 사용자 정의 방법을 자세히 살펴보았습니다. Highcharts를 효과적으로 사용하여 데이터 시각화를 개선하고 사용자 경험을 향상시킬 수 있습니다. <br/>
<br/>
위의 예제들은 highcharts의 jsfiddle에 있는 코드를 참조하여, 직접 typescript 버전으로 수정한 코드입니다. <br/>
추가 질문이나 코드 수정에 관한 문의가 있으면 언제든지 댓글이나 이메일로 문의해 주시기 바랍니다.<br/>



