---
title: "Highcharts Label - Changing Attributes Dynamically)"
date: 2023-09-26 13:49:00 +0900
comments: true
categories: angular
tags: [highcharts, highcharts-angular]
---

> 이 글에서는 Highchart에서 label을 사용할 때 style을 변경하는 방법에 대해 알아보겠습니다.


## 분석
label의 attribute를 변경하는 것은 일반적인 방법으로는 중간에 값을 변경하기 어려습니다.

이를 확인하기 위해 label의 html 구조를 보면 label은 <g>태그와, 별도의 <div>가 조합된 것을 볼 수 있습니다.

만일 component에서 label의 이벤트를 세팅하여 lable을 표시하는 dom을 가져오려고 시도하면 <g> 태그가 잡히게 되는데
문제는 우리가 원하는 attribute의 설정은 <div> 태그와 관련이 있다는 점 입니다.

이러한 문제를 해결하고자 몇가지 편법을 사용하려고 합니다.

먼저 지난 시간에 사용했던 customized tooltip을 예제로 참고하여 확장해봅시다.
기존 customized tooltip 은 아래와 같습니다.

```tsx
  this.chart = Highcharts.chart({
    ...
    tooltip: {
      enabled: false // disable the default one
    },
    plotOptions: {
      series: {
        point: {
          events: {
            click: (p: any) => {
              this.chart.renderer.label(
                'tooltip title', // title
                p.offsetX, // x position
                p.offsetY, // y position
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

## Set Attributes on Label at the Beginning
label에 style 넣으려면 html 형태의 데이터가 들어가야 합니다.
따라서 text를 html로 변경해야 합니다.

### Set useHTML
또한, html을 사용가능 상태로 설정해야 합니다. 이 옵션은 renderer.label의 옵션에서 제공됩니다.
```
SVGRenderer.label(str: string, x: number, y?: number, shape?: string, anchorX?: number, anchorY?: number,
useHTML?: boolean, baseline?: boolean, className?: string): Highcharts.SVGElement
```
label 전체에 class를 적용하려면 className 옵션을 설정하고, label의 각 항목에 class 또는 style을 적용하려면 useHTML을 설정합니다.
여기에서는 useHTML을 활용해보겠습니다.

위의 예제를 다음과 같이 변경할 수 있습니다.
```tsx
  ...
  this.chart.renderer.label(
    `<p>Title</p><button type="button" class="btn">Click to Confirm<button>`,
    p.offsetX, 
    p.offsetY, 
    null, null, null, true
  )
  ...

```

### Store to the Variable
설정한 label을 변수에 저장해야 합니다.
변수에 저장하는 이유는 dom 의 접근 때문인데 이후에 다시 알아보도록 합시다.

```tsx
let label =   
  this.chart.renderer.label(
    `<p>Title</p><button type="button" class="btn">Click to Confirm<button>`,
    p.offsetX, 
    p.offsetY, 
    null, null, null, true
  );
```

### Set Event
만일 label에 event가 필요하다면 on 함수를 사용하여 정의할 수 있습니다.
만일, label을 click하였을 때 label의 class를 변경한다고 가정한다면, click event를 정의해야 합니다.

label을 click하면 class명을 추가하는 코드를 작성해 봅시다.

click event는 다음과 같이 설정할 수 있습니다.
```tsx
label.on('click', (event: any) => {

});
```

이 때 주의해야할 점은 event가 target으로 삼는 dom은 <g> 라는 점입니다. 
우리가 활용할 값은 바로 변수 label의 element 입니다. 이 값은 <div>에 속해 있는데 이 값을 통해 class를 변경할 수 있습니다.
또한 div 안의 버튼이나 기타 dom에도 접근할 수 있습니다.

```tsx
label.on('click', (event: any) => {
  label.element.addClass('active'); // example of setting class to label
  label.element['div'].children[0].childNodes[0].style.color = 'red'; // example of setting firstNode style
});
```






