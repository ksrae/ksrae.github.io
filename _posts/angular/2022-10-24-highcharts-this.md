---
title: "How to Use This of Highchart in Typescript"
date: 2022-10-21 13:49:00 +0900
comments: true
categories: angular
tags: [highcharts, highcharts-angular]
---

예제들을 검색하다보면 javascript 또는 jquery로 된 예제들을 볼 수 있는데<br/>

그런 예제들 중에 this로 처리되어 있는 예제들을 간혹 확인할 수 있습니다.<br/>
<br/>
하지만 typescript나 angular 에서의 this는 그 의미가 다르므로 그대로 코드를 옮기면 에러가 발생합니다.<br/>
<br/>
이 글에서는 그러한 에러를 어떻게 해결할 수 있는지 알아보겠습니다.<br/>

## Example of this in javascript

구글에 검색해보면 highcharts에서 this의 의미가 무엇인지에 대한 많은 질문들을 확인할 수 있습니다.<br/>
<br/>
예를 들면 다음과 같은 경우를 볼 수 있습니다.<br/>

```js
tooltip: {
  formatter: function() {
    return '<b>'+ (this.point ? this.point.name : this.series.name)  +'</b><br/>';
  }
},
```

typescript나 angular에서는 이를 어떻게 적용할 수 있을까요?<br/>
위의 코드에서 formatter는 파라미터가 없는 함수로 되어 있는데 typescript나 angular에서는 파라미터를 받아올 수 있습니다.<br/>
이 파라미터 값이 바로 위의 this를 대응하는 값이므로 위의 코드는 아래와 같이 수정하여 적용할 수 있습니다.<br/>

```tsx
  formatter: (e: any) {
    return '<b>'+ (e.point ? e.point.name : e.series.name)  +'</b><br/>';
  }
```

위의 예제에서는 편의상 any 타입으로 선언하였으나 Highcharts의 API 문서를 참조하면 정확한 타입을 확인할 수 있습니다.<br/>
예를 들어 위의 formatter에서의 this의 type은 ([링크 참조](https://api.highcharts.com/class-reference/Highcharts#.TooltipFormatterCallbackFunction))Highcharts.TooltipFormatterContextObject 이므로 다음과 같이 수정할 수 있습니다.

```tsx
  formatter: (e: Highcharts.TooltipFormatterContextObject) {
    return '<b>'+ (e.point ? e.point.name : e.series.name)  +'</b><br/>';
  }
```



## Reference
[Highcharts-JS API Reference](https://api.highcharts.com/highcharts/)
