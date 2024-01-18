---
title: "Highcharts-angular oneToOne 옵션으로 쉽게 데이터 업데이트 하기 - easy update with oneToOne Option"
date: 2022-11-28 18:11:00 +0900
comments: true
categories: angular
tags: [highcharts]
---

Highchart는 데이터를 수정하기 위한 다양한 옵션을 가지고 있고, 관련 api 문서도 잘 구성되어 있기 때문에 highcharts-angular를 사용함에 있어서도 큰 어려움이 없습니다.<br/>
다만 모든 상황에서 모든 명령을 일일이 찾기란 번거로운 일이며, 아마도 값을 주입하는대로 적용되는 것이 가장 쉽고 빠른 방법일 것입니다.<br/>
<br/>
이 글에서는 options을 변경하는 즉시 적용하는 방법으로 차트를 구성하는 방법을 알아보겠습니다. <br/>

## 일반적인 Chart 구현 방식
일반적으로 우리는 차트를 다음과 같이 구현합니다.<br/>

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

여기에 series를 추가하고 싶다면, api 문서를 통해 이를 확인하고 해당 함수를 아래와 같이 적용할 수 있습니다.<br/>

```tsx
this.Highcharts.addSeries({
  data: [1,2,3,4,5,6,7,8,9,0]
});

```

이는 모든 api를 알아야하는 불편함이 있습니다. 따라서 이를 개선하기 위해 options을 직접 수정하는 방법을 알아보겠습니다.<br/>


## 개선된 Chart 구현 방식
놀랍게도 chartOptions을 직접 수정해도 chart에 적용됩니다. 만일 chart의 type을 bar로 변경하고 싶다면, 간단하게 다음과 같이 수정하면 됩니다.<br/>

```tsx
this.chartOptions = {
  ...this.chartOptions,
  chart: {
    ...this.chartOptions.chart,
    type: 'bar'
  }
}
```

하지만 위의 예시과 같이 series를 변경하면 적용되지 않습니다. 이는 바로 옵션 onetoOne의 default값이 false이기 때문입니다. ([이전 글 참조](/highcharts-angular)) <br/>
<br/>
하지만, 아래와 같이 <b>onetoOne을 true로 설정하고, series를 변경하면 적용됨을 확인</b>할 수 있습니다.<br/>

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

이 방식의 장점은 api를 매번 확인하지 않아도 된다는 점입니다. 따라서 여러 옵션들을 빠르고 쉽게 변경할 수 있습니다.






