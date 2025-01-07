---
title: "Control flow - If, Switch and For"
date: 2025-01-07 10:41:00 +0900
comments: true
categories: angular
tags: [controlflow, ngif, ngfor]
---

Angular는 강력한 프레임워크로, 개발자들이 효율적으로 웹 애플리케이션을 구축할 수 있도록 다양한 기능을 제공하는데 그 중 하나가 바로 Control Flow입니다. <br/>
이번 글에서는 Control Flow가 무엇인지, 어떻게 사용되는지, 그리고 그로 인해 얻는 이점에 대해 자세히 알아보겠습니다.

# Control Flow란?
Control Flow는 Angular 템플릿 내에서 조건문과 반복문을 더 직관적이고 간결하게 작성할 수 있도록 해주는 기능입니다. <br/>
이전에는 ngFor, ngIf와 같은 디렉티브를 사용하여 DOM을 조작했으나, 이 방식은 코드가 복잡해지고 가독성이 떨어지는 단점이 있었습니다. <br/>
특히, ngFor와 ngIf를 함께 사용할 수 없어 불필요한 DOM 추가와 복잡한 구조가 발생했습니다.<br/><br/>

Control Flow의 도입으로 코드와 HTML이 분리되어 가독성이 향상되었고, CommonModule을 import하지 않아도 되어 코드 작성이 더욱 편리해졌습니다. <br/>
뿐만 아니라, 기존 CommonModule 대비 처리 속도가 약 45% 개선되었습니다.

## 예제

### ngIf를 사용했을 때,

```ts
@Component({
	standalone: true,
	template: `
	<div *ngIf="isLoggedin">
		Welcome back, user!
	</div>`,
	imports: [
		NgIf // or CommonModule
	],
})
export class AppComponent {}

```

### control flow를 사용했을 때,

```ts
@Component({
	standalone: true,
	template: `
	@if (isLoggedin) {
		<div>
			Welcome back, user!
		</div>
	}`,
})
export class AppComponent {}

```

## @if
@if는 주어진 조건에 따라 콘텐츠를 조건부로 렌더링하는 기능입니다. 이 기능은 특정 조건이 참인지 거짓인지에 따라 다른 HTML 블록을 표시할 수 있게 해줍니다.<br/>
이로 인해 코드의 가독성이 높아지고, 불필요한 DOM 요소를 줄일 수 있습니다.<br/>
아래의 예제에서 condition이 true일 경우 첫 번째 <div>가 렌더링되고, false일 경우 두 번째 또는 세 번째 <div>가 렌더링됩니다.

```html
{% raw %}
@if(condition) {
	<div>Content</div>
} @else if (!condition) {
	<div>Else If Content</div>
} @else {
	<div>Else Content</div>
}
{% endraw %}

```

## @switch
@switch는 여러 조건을 다루는 데 유용한 기능으로, 복잡한 조건문을 간결하게 작성할 수 있게 해줍니다.<br/>
여러 케이스를 정의하고, 주어진 조건에 따라 해당 케이스에 맞는 콘텐츠를 렌더링합니다.<br/>


```html
{% raw %}
@switch(condition) {
	@case ('a') {
		<div>A</div>
	} @case ('b') {
		<div>B</div>
	} @default {
		<div>Others</div>
	}
}
{% endraw %}
```

## @for
@for는 리스트를 반복하여 각 항목을 처리하는 기능입니다. 이 기능을 통해 동적인 데이터를 쉽게 렌더링할 수 있습니다.<br/>
반복문을 사용함으로써 코드 중복을 줄이고, 데이터가 변경될 때 유연하게 대처할 수 있습니다.

```html
{% raw %}
@for(item of list) {
	<li>{{item.value}}</li>
}
{% endraw %}
```

### track

track은 ngFor의 trackBy 기능과 유사하여, 리스트의 각 항목을 효율적으로 추적할 수 있게 해줍니다.<br/>
이 기능을 활용하면 Angular는 각 항목의 고유 ID를 기반으로 DOM 요소를 관리합니다.

```html
{% raw %}
@for(item of list;track item.id) {
	<li>{{item.value}}</li>
}
{% endraw %}
```

track은 또한 내장 variable을 활용한 확장이 가능합니다.

| $index | row index |
| --- | --- |
| $first | first row |
| $last | last row |
| $even | 짝수 row |
| $odd | 홀수 row |


```html
{% raw %}
@for(item of list; track trackId($index, item)) {
	<li>{{item | json}}</li>
}
{% endraw %}
```
이렇게 하면 trackId 함수를 통해 track을 설정할 수 있습니다.

### 내장 변수 활용
@for에서는 기본 변수를 활용하여 각 항목에 대한 추가 정보를 제공할 수 있습니다.<br/>
아래의 예제에서는 $index, $first, $last, $even, $odd와 같은 내장 변수를 사용하고 있습니다.

```html
{% raw %}
@for(item of list;let index = $index, let first = $first, let last = $last, let even = $even, let odd = $odd) {
	<li>
		{{index}}
		{{first}}
		{{last}}
		{{even}}
		{{odd}}
	</li>
}
{% endraw %}
```

## @empty
@empty는 반복문을 수행했을 때 리스트가 비어 있는 경우에 실행되는 블록입니다. <br/>
리스트의 길이가 0이거나 undefined인 경우에 사용자에게 적절한 메시지를 표시할 수 있습니다.

```html
{% raw %}
@for(item of list) {
	<li>{{item | json}}</li>
} @empty {
	No Item
}
{% endraw %}
```