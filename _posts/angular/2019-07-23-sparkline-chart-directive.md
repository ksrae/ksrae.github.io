---
title: "Creating Sparkline Chart Directive with ng2-nvd3"
date: 2019-07-22 14:42:00 +0900
comments: true
categories: javascript
tags: [sparkline, chart, directive, nvd3, ng2-nvd3]
---


보통의 차트 라이브러리가 Sparkline Chart를 지원하지 않아 (특히 Angular를 지원하는 차트 라이브러리) 직접 만들어 사용해보려고 합니다.
활용할 차트 라이브러리를 이것 저것 보다가 ng2-nvd3로 그린 차트가 가장 마음에 들어서 이 라이브러리를 활용하여 만들어 보겠습니다.
손쉬운 재활용을 위해 Directive로 제작하였습니다. 


## 필요사항
nvd3를 활용하되 angular 에서 사용하려면 ng2-nvd3를 install 해야합니다.

    npm install ng2-nvd3


## 특징
- Sparkline Chart는 하나의 라인에 기준점을 기준으로 상, 하에 각기 다른 색을 지정할 수 있는 차트를 의미합니다. 일반 라인 차트는 이러한 기능을 제공하지 않습니다.
- 같은 차트 2개를 겹쳐 그리고 한쪽 차트의 칠하는 범위를 반전한 뒤 기준점을 기준으로 잘라내는 방법을 사용합니다.
- nvd3 명령을 사용하지 않고 dom에 직접 svg를 그려보려고 합니다.


## Directive params
Directive 호출할 때 사용할 옵션 입니다. default가 없는 파라미터는 반드시 채워 넣어야 합니다.

```html
{% raw %}
<div id="sparklineThreshold"
        [sparklineThreshold]="sparklineThresholddata"  
        [height]="100" 
        [width]="400" 
        [type]="linear" 
        [threshold]="100" 
        [strokecolors]="['red','blue']" 
        [fillcolors]="['#da343452', '#c7daea']">
</div>
{% endraw %}
```

### 설명
- id:  여러 directive를 한 페이지에서 동시에 활용하기 위해 dom은 반드시 id를 가지도록 설계하였습니다.
- sparklineThreshold: directive 이름 입니다, 여기에 chart data를 전송하면 됩니다. 데이터는 Array\<number>로 구조로 보냅니다.
- height: svg의 높이
- width: svg의 길이. width를 기준으로 x축을 데이터 길이에 따라 나누어 만들어 냅니다.
- type: linear와 basis 두 가지가 있는데 default는 linear를 적용합니다. basis는 곡선으로 그려주는 옵션입니다.
- threshold: 기준점. 상, 하 차트를 그리는 기준점이 되므로 이 기준에 따라 색의 적용 범위가 결정됩니다.
- strokecolors: 상, 하 차트의 선 색. default는 'red', 'blue'
- fillcolors: 상, 하 차트의 배경색. default는 '#da343452', '#c7daea'

### 데이터 구성
- threshold: min, max값을 넘지 않도록 하고 이 값을 비율로 계산합니다.

```ts
this.maxData = Math.max(...this.rawData);
this.minData = Math.min(...this.rawData);

// threshold가 max를 넘지 않는 범위에서 퍼센트로 변경
let threshold = (this.rawThreshold > this.maxData) ? this.maxData : this.rawThreshold;
threshold = (this.rawThreshold < this.minData) ? this.minData : this.rawThreshold;
this.threshold = Math.round((1 - (this.maxData - threshold) / this.maxData) * 100);
```

- x축: width를 기준으로 구성.
- y축: 최대값을 기준으로 비율로 구성.

```ts
const dataLength = this.rawData.length;
const step = Math.round(this.width / this.rawData.length);
let i = 0;

for (const item of this.rawData) {
    this.data.push({
        x: i, y: Math.round((1 - (this.maxData - item) / this.maxData) * 100)
    });
    i += step;
}
```

### 그래프 그리기
- 동일한 두 개의 그래프를 그립니다.
- domain을 min 부터 max 데이터까지 범위를 설정합니다.
- range를 height 부터 0까지 범위를 설정합니다.

#### 상한가 그래프
- clipPath의 y를 0. height를 scale된 threshold 로 설정합니다.
- path에서 clipPath를 설정합니다.
- clipPath에 의해 위쪽 데이터만 구현됩니다.

#### 하한가 그래프
- clipPath의 y를 scale된 threshold 로 설정하고, height는 maxData로 설정합니다.
- path에서 clipPath를 설정합니다. 
- clipPath에 의해 아래쪽 데이터만 구현됩니다.

## 전체소스
```ts
import { Directive, Input, ViewContainerRef, AfterViewInit } from '@angular/core';
declare let d3: any;

@Directive({
    selector: '[sparklineThreshold]'
})

export class SparkLineThresholdDirective implements AfterViewInit {
    rawData = [];
    data = [];
    maxData = 0;
    minData = 0;
    maxWidth = 0;
    rawThreshold = 0;
    threshold = 0;
    topStrokeColor = 'red';
    bottomStrokeColor = 'blue';
    topFillColor = '#da343452';
    bottomFillColor = '#c7daea';
    elementId;

    // 실 데이터 = [ 숫자 ] 형태
    @Input('sparklineThreshold') set setData(data: Array<number>) {
      this.rawData = data;
      
      // 여기는 테스트
     // const numPoints = 50;
     //   for (let i = 1; i < numPoints; i++) {
     //       const rnd = Math.floor(Math.random() * (250)) + 1;
     //      this.rawData.push(rnd);
      //  }
    }

    @Input('width') width: number;
    @Input('height') height: number;
    @Input('type') type: string;
    @Input('threshold') set setThreshold(threshold: number) { this.rawThreshold = threshold; }
    @Input('strokecolors') set setStrokeTopColor(colors: Array<string>) {
        this.topStrokeColor = colors[0];
        this.bottomStrokeColor = colors[1];
    }
    @Input('fillcolors') set setFillColors(colors: Array<string>) {
        this.topFillColor = colors[0];
        this.bottomFillColor = colors[1];
    }

    constructor(private viewContainer: ViewContainerRef) {
        this.elementId = this.viewContainer.element.nativeElement.id;
    }

    // rawdata가 있어야 하고, id가 있어야 한다.
    ngAfterViewInit() {
        if (!this.rawData || !this.rawData.length) {
            console.log('rawData are required');
            return;
        }
        if (!this.elementId) {
            console.error('element Id is required');
            return;
        }

        // 초기값
        this.maxData = Math.max(...this.rawData);
        this.minData = Math.min(...this.rawData);
        // threshold가 max를 넘지 않는 범위에서 퍼센트로 변경
        let threshold = (this.rawThreshold > this.maxData) ? this.maxData : this.rawThreshold;
        threshold = (this.rawThreshold < this.minData) ? this.minData : this.rawThreshold;
        this.threshold = (100 - Math.round(((this.maxData - threshold) / this.maxData) * 100));

        this.type = this.type ? this.type : 'linear';
        this.setSparklineData();
        this.drawSparkLine();

        // resize https://github.com/novus/nvd3/issues/645
    }

    // rawData를 mapping 한다.
    setSparklineData() {
        const dataLength = this.rawData.length;
        const step = Math.round(this.width / this.rawData.length);

        let i = 0;
        for (const item of this.rawData) {
            this.maxWidth = step * i;

            this.data.push({
                x: this.maxWidth, y: (100 - Math.round(((this.maxData - item) / this.maxData) * 100))
            });

            i++;
        }
    }

    // 그리자
    drawSparkLine() {
        const svg = d3.select(this.viewContainer.element.nativeElement).append('svg');
        svg.attr('width', this.width).attr('height', this.height + 5); 
        // 안보이는 margin이 있어서 5 정도 여유를 두어야 한다. 
        // 안그러면 끝점이 가려지는 부분이 생긴다.

        const draw = svg.append('g');

        const yScale = d3.scale.linear()
        .domain([this.minData, this.maxData])
        .range([this.height, 0]);

        // top
        const setTopLine = d3.svg.area()
        .x((d) => d.x)
        .y((d) => yScale(d.y))
        .y0(this.height)
        .interpolate(this.type);

        draw.append('clipPath')
        .attr('id', this.elementId + '-top')
        .append('rect')
        .attr('x', 1) // area라 0점에서 시작하기 때문에 좌측 시작점을 가리기 위해 1을 준다.
        .attr('y', 0)
        .attr('rx', 0)
        .attr('ry', 0)
        .attr('height', yScale(this.threshold))
        .attr('width', this.maxWidth - 2); // area라 끝점에서 0으로 수렴하는 부분을 가리기 위해 2를 뺀다.

        draw.append('path')
        .data([this.data])
        .attr('d', setTopLine)
        .attr('clip-path', 'url(#' + this.elementId + '-top)')
        .attr('stroke', this.topStrokeColor)
        .attr('stroke-width', 1)
        .attr('fill', this.topFillColor);

        // bottom
        const setBottomLine = d3.svg.area()
        .x((d) => d.x)
        .y((d) => yScale(d.y))
        .y0(this.threshold)
        .interpolate(this.type);

        draw.append('clipPath')
        .attr('id', this.elementId + '-bottom')
        .append('rect')
        .attr('x', 1) // area라 0점에서 시작하기 때문에 좌측 시작점을 가리기 위해 1을 준다.
        .attr('y', yScale(this.threshold))
        .attr('rx', 0)
        .attr('ry', 0)
        .attr('height', this.maxData)
        .attr('width', this.maxWidth - 2); // area라 끝점에서 0으로 수렴하는 부분을 가리기 위해 2를 뺀다.

        draw.append('path')
        .data([this.data])
        .attr('d', setBottomLine)
        .attr('clip-path', 'url(#' + this.elementId + '-bottom)')
        .attr('stroke', this.bottomStrokeColor)
        .attr('stroke-width', 1)
        .attr('fill', this.bottomFillColor);
    }
}
```

이상입니다.

