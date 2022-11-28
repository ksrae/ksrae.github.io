---
title: "What is charts variables of Highcharts in Highcharts-angular?"
date: 2022-11-15 13:49:00 +0900
comments: true
categories: angular
tags: [highcharts, highcharts-angular]
---

Highcharts에서는 chart 변수에 직접 options을 주입하기 때문에 api 의 활용은 물론, 값의 수정도 쉽게 적용할 수 있습니다.<br/>
하지만 highcharts-angular에서는 chart 변수에 직접 적용하려고 하면 다른 상황을 겪게 됩니다.<br/>
<br/>
이 글에서는 highcharts-angular에서 highcharts와 같이 charts 변수에 접근하는 방법을 알아보겠습니다.<br/>

## Highcharts 에서 chart 변수 접근하기

Highcharts에서는 아래와 같이 chart 변수에 직접 Highchart.chart를 주입합니다. <br/>

```tsx
export HighchartsComponent implements OnInit {
  chart: any;

  ngOnInit() {
    this.chart = Highchart.chart({
      ...
    })
  }
}
```

따라서 chart 변수를 직접 접근하면 되므로 어렵지 않습니다.<br/>

```tsx
const series = this.chart.series;
```


## Highcharts-angular 에서 chart 변수 접근하기

Highcharts-angular에서는 chart 변수에서 options에 접근하려면 chart 변수값이 Highcharts와는 다른 구조로 되어 있어 매우 복잡해 보입니다. 또한 눈에 보이는 Chart 변수에 직접 접근을 시도하면 제약이 많아 불편합니다.<br/>

## 해결 방안 1 - Highcharts.charts[0]

답은 바로 눈에 보이는 Chart 변수가 아닌 배열로 구성된 charts[] 변수에 접근 하는 것입니다. 차트가 하나라면 chart[0]에 접근할 수 있으며, 이제 비로소 highcharts에서의 chart 변수값과 같은 값을 가짐을 확인할 수 있습니다.<br/>
<br/>
즉, highcharts의 this.chart 는 highcharts-angular의 this.Highcharts.charts[0]과 같다고 할 수 있습니다.<br/>

```tsx
const series = this.Highcharts.chart[0].series;
```


## 해결 방안 2 - callbackFunction

위의 방법의 단점은 chart의 index를 알아야한다는 점 입니다. 즉, index를 모르면 잘못된 chart에 명령을 내릴 수 있는 여지가 있습니다. 반면에 callbackFunction은 파라미터 값이 곧 chart 이므로 사용하기 쉽다는 장점이 있습니다.<br/>
<br/>
callbackFunction은 chart 생성 시 자신의 chart 변수를 전달해줍니다. 이는 반드시 컨트롤하는 chart를 의미하므로 실수할 여지가 없고, chart 생성 시점에 값을 받으므로 명령이 생성 이전에 주입될 걱정도 덜 수 있습니다.<br/>
<br/>
작성 방법은 다음과 같습니다. 이 예제에서는 callback을 받아 상위로 전달하는 코드를 작성해보았습니다.<br/>

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

주의:
이 방식의 가장 큰 문제점은 chart 명령어를 통해 print 기능이나 또는 export image 기능을 활용할 경우 1회만 실행이 가능하다는 것입니다. 여러번 이상 실행할 경우 chart값이 undefined 라는 오류를 일으키며, 동작하지 않습니다.


## 해결 방안 3 - chartInstance

highchart-angular는 생성 시 chartInstance를 만듭니다. 이 값을 output을 활용해 외부로 전달하여 사용하는 방식입니다.
이 방식은 위의 callback과 동일한 기능을 제공하지만 callback에서 발견된 문제점도 해결할 수 있습니다.
즉, print나 export 기능을 중복 실행해도 문제없이 수행이 가능하다는 점입니다.

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

