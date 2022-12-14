---
title: "ngFor with TrackBy"
date: 2020-07-08 14:25:00 +0900
comments: true
categories: angular
tags: [ngfor, trackby]
---


ngFor는 DOM을 반복해서 출력해주는 기능을 합니다. 여기에 몇가지 추가 요소를 붙일 수 있는데 대표적인 것으로는 index가 있습니다.

```html
{% raw %}
<div *ngFor="let item of items;let idx=index;"></div>
<!-- 또는 -->
<div *ngFor="let item of items;index as idx;"></div>
{% endraw %}
```

ngFor의 단점은 for문 안에 영향을 미치는 이벤트가 발동되면 모든 Dom이 refresh 된다는 점입니다. 즉, 많은 돔이 물려 있을수록 더 많은 부하가 걸릴 수 있습니다. 

## 사용법

trackBy를 활용하면 이벤트에 해당하는 DOM만 refresh가 발동합니다. 단, 이 때, trackBy에는 primary 값. 다시 말해서 고유값만 포함하여야 합니다. 사용법은 아래와 같습니다.

```tsx
let items = [
	{ id: 1, name: 'first'},
	{ id: 2, name: 'second'},
	{ id: 3, name: 'third'},
	{ id: 4, name: 'fourth'}
];

trackByItem = (item): number => item.id;
```

위와 같은 데이터가 있다고 할 때, trackByItem이라는 추적할 대상(trackBy)을 리턴하는 함수를 만들어 둡니다. 그리고, Template에서는 아래와 같이 작성합니다.

```html
{% raw %}<div *ngFor="let item of items;trackBy: trackByItem"></div>{% endraw %}
```

이제 for문 내의 데이터는 오직 이벤트에 해당하는 DOM만 refresh 됩니다.

## 추가
Angular 버전에 따라 아래의 함수를 작성할 경우 에러가 발생할 수 있습니다.

```
trackByItem = (item): number => item.id;
```

이는 파라미터로 index를 받도록 변경되었기 때문이므로 파라미터를 item 외에 index를 함께 받아주어 해결할 수 있습니다.

```
trackByItem = (index: number, item: any): number => item.id;
```

## 참고 사이트
- [https://angular.io/guide/template-syntax#ngfor-with-trackby](https://angular.io/guide/template-syntax#ngfor-with-trackby)
- [https://stackoverflow.com/questions/42108217/how-to-use-trackby-with-ngfor](https://stackoverflow.com/questions/42108217/how-to-use-trackby-with-ngfor)