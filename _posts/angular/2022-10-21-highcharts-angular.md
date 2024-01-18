---
title: "How to setup highcharts-angular"
date: 2022-10-21 13:49:00 +0900
comments: true
categories: angular
tags: [chart, highcharts]
---

Highcharts-Angular는 Highcharts를 Angular에서 사용하기 쉽도록 작성된 library 입니다.<br/>
<br/>
이 글에서는 Highcharts와 Highcharts-Angular를 설치하는 방법을 알아보고,<br/>
Highchart-Angular에서 제공하는 기본 Input에 대하여 정리하겠습니다.<br/>
<br/>

## 왜 Highcharts-Angular를 사용하는가?
Highcharts는 사실 다른 library를 추가로 설치하지 않아도 Angular 환경에서 충분히 사용할 수 있습니다.<br/>
예를 들어 Angular에서 다음과 같이 작성하면 highcharts-angular의 도움 없이도 충분히 사용이 가능합니다.<br/>

```html
<div id="chart-line"></div>
```

```tsx
import * as Highcharts from 'highcharts';

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

### 그러면 왜 highcharts-angular를 활용하는 것이 좋을까요?
- zone.js의 이벤트를 handling할 수 있습니다. zone의 도움 없이 강제로 이벤트를 fire 하거나 또는 로직을 수행하고 angular outside에 태워 이벤트를 강제로 태우지 않을 수도 있습니다.<br/>
- option만 주입하는 간단한 형태로 코드를 구성할 수 있습니다. 즉, 이전에는 Highcharts.chart를 통해 인스턴스를 생성하고, 이를 컨트롤했다면 highcharts-angular는 옵션만 해당 component에 주입하면 적용되는 형태로 간단해졌습니다. 만일 기존 코드를 동일하게 구성하려면 observable 패턴을 구성해야 하는 번거로움이 있습니다.<br/>
- 원한다면 highcharts-angular에서 chart 인스턴스를 output하여 활용할 수도 있습니다. 차트를 작성하는 코드 구성은 간단해지면서도 모든 기능을 다룰 수 있는 형태를 갖추고 있습니다.<br/>
- chart의 명령을 몰라도 옵션을 컨트롤하여 chart의 대부분의 기능을 사용할 수 있습니다. 관련해서는 다음에 보다 더 자세히 다룰 예정입니다.<br/>


## How to Set-up

당연한 말이겠지만 highcarts-angular를 사용하기 위해서는 반드시 highcharts를 install 해야 합니다.<br/>
또한 highcharts-angular는 module을 반드시 import 해야 합니다.<br/>

```
npm i highcharts
npm i highcharts-anguilar
```

```tsx
import { HighchartsChartModule } from 'highcharts-angular'
...
@NgModule({
   imports: [
      HighchartsChartModule
   ]
})
```

## How to Use

### Template
highcharts-angular에서 제공하는 기본 옵션은 다음과 같습니다.<br/>

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

Attribute를 살펴 보면 다음과 같습니다.<br/>
- `highcharts: typeof Highcharts // required`, highcharts 변수를 전달해야 합니다.<br/>
- `chartConstructor: string = 'chart'; // optional`, chart의 종류를 정의 합니다. 가능한 옵션은 'chart', 'stockChart', 'mapChart', 'ganttChart' 입니다.<br/>
- `chartOptions: Highcharts.Options = { ... }; // required`, 차트를 구성하는 값을 전달해야 합니다.<br/>
- `chartCallback: Highcharts.ChartCallbackFunction = function (chart) { ... } // optional`, 생성된 차트의 callback 함수입니다.<br/>
- `updateFlag: boolean = false; // optional`, chart 생성 후 조작할 때 update를 true로 주면 강제로 trigger 할 수 있습니다. true 값은 trigger가 적용되면 false로 변경됩니다.<br/>
- `oneToOneFlag: boolean = true; // optional`, seires, xAxis, yAxis를 추가 또는 제거할 때 사용합니다. update와는 다르게 기존 값을 입력된 값으로 교체하지 않고, 주입된 값을 기존 값에 추가 또는 제거하는 방식으로 적용합니다.<br/>
- `runOutsideAngular: boolean = false; // optional`, 강제로 not trigger 됩니다. 이후 updateFlag를 통해 한꺼번에 trigger를 시도하면 좋은 효율을 가질 수 있습니다. <br/>


### Component

위에서 required로 전달할 값을 정의하는 법을 알아보고, highcharts를 import 하는 방법을 알아봅시다.<br/>
highcharts는 별도의 d.ts를 제공하지 않으므로 declare된 모든 값을 import 해야 합니다.<br/>


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

## Loading Module / Plugin
highcharts는 추가 모듈을 제공하는데 이를 typescript 또는 angular 에서 import하려면 일반적이지 않은 방법을 적용해야 합니다. 해당 module을 import한 뒤 import한 모듈에 정의한 chart 변수를 주입해야 합니다. <br/>
Plugin도 방식은 동일합니다.<br/>


```tsx
import HC_exporting from 'highcharts/modules/exporting';
HC_exporting(Highcharts);
```


## SSR
ssr에서는 highchart가 로드되기 전에 호출되는 것을 방지하기 위해 chart 변수에 값이 정의되기 전에 화면에 노출되지 않도록 ngIf를 추가해야 합니다. <br/>
그렇지 않으면 아직 값이 적용되지 않은 변수를 highcharts-angular에 주입을 시도하므로 에러가 발생합니다.

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
