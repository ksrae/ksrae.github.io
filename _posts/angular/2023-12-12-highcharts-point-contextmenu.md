---
title: "Setting Contextmenu Event to all Points in a series of Highcharts without  highcharts-custom-events"
date: 2023-12-12 14:08:00 +0900
comments: true
categories: angular
tags: [highcharts, highcharts-angular, contextmenu]
---


Highcharts에서는 points에 contextmenu를 기본 event로 제공하지 않습니다.<br/>
그래서 일반적으로 npm에서 [highcharts-custom-events](https://www.npmjs.com/package/highcharts-custom-events) 를 설치하는 방법을 추천합니다.

그러나 이번 블로그에서는 3rd party없이 Highcharts 차트에서 모든 데이터 포인트에 contextmenu event를 적용하는 방법에 대해 알아보겠습니다.

## 해결책

### 1. Chart 생성 시 이벤트 핸들러 등록
Highcharts 차트를 생성할 때, load 이벤트를 활용하여 각 시리즈의 모든 데이터 포인트에 대한 컨텍스트 메뉴 이벤트 핸들러를 등록합니다. 이때, Angular 컴포넌트의 메소드를 활용하여 컨텍스트 메뉴 이벤트를 처리합니다.

```ts
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


### 2. 컨텍스트 메뉴 구현
컨텍스트 메뉴를 구현할 때는 preventDefault를 사용하여 브라우저 기본 동작을 막고, 필요한 사용자 정의 동작을 정의합니다. 위의 코드에서는 단순히 콘솔에 로그를 출력하도록 하였지만, 실제 프로젝트에서는 여러 가지 동작을 추가할 수 있습니다.


### 3. allowPointSelect 설정

point event를 받으려면 반드시 모든 series의 allowPointSelect 는 true 로 설정되어야 합니다.
한꺼번에 적용하려면 plotOptions을 활용하는 것이 좋습니다.

```ts
  plotOptions: {
    series: {
      allowPointSelect: true,
    }
  }
```


## 마무리
이렇게 구현된 Highcharts 차트는 추가 라이브러리의 설치 없이 각 데이터 포인트에 사용자 정의된 컨텍스트 메뉴를 제공하게 됩니다. 이는 사용자가 시각화된 데이터에 대한 추가적인 정보나 조작을 손쉽게 할 수 있게 해주어 전반적인 사용자 경험을 향상시킵니다.


## Example Codes
```ts
import { ChangeDetectionStrategy, ChangeDetectorRef, Component, OnInit } from '@angular/core';
import * as Highcharts from 'highcharts';

@Component({
  selector: 'app-contextmenu',
  template: `<div id="chart-line"></div>`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ContextMenuComponent implements OnInit {
  chart: any;

  ngOnInit(): void {
    this.createChartLine();
  }

  private createChartLine(): void {
    this.chart = Highcharts.chart({
      chart: {
        renderTo: 'chart-line',
        type: 'line',
        borderWidth: 1,
        plotBorderWidth: 1,
        marginRight: 50,
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
      },

      xAxis: {
        categories:  ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];,
      },

      series: [{
          data: [29.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5, 216.4, 194.1, 95.6, 54.4],
      }],

      plotOptions: {
        series: {
          allowPointSelect: true,
          point: {
            events: {

            }
          },
          marker: {
            enabled: true,
            fillColor: 'blue',
            lineWidth: 2,
            symbol: 'square',
            states: {
              // https://api.highcharts.com/highcharts/plotOptions.line.marker.states.select
              select: {
                enabled:true,
                fillColor:'blue',
                lineColor:'red',
                lineWidth:2,
                radius:undefined
              }
            }
            // lineColor: Highcharts?.getOptions().colors[0]
          },
        },

      },

    } as any)
  }

}

```