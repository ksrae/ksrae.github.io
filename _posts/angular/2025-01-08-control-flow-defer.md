---
title: "Control flow - defer"
date: 2025-01-08 10:41:00 +0900
comments: true
categories: angular
tags: [controlflow]
---

## @defer

지연 가능한 뷰, 즉 @defer 블록은 페이지의 초기 렌더링에 엄격히 필요하지 않은 코드의 로딩을 지연시켜 애플리케이션의 초기 번들 크기를 줄입니다.<br/>
이로 인해 초기 로드 속도가 빨라지고 Core Web Vitals(CWV), 특히 Largest Contentful Paint(LCP)와 Time to First Byte(TTFB)가 개선되는 경우가 많습니다.

### 주요 개념

- 사용자 경험 최적화: 현대 웹 디자인에서는 페이지 로딩 중 사용자 경험을 최적화하는 것이 중요합니다. 사용자가 페이지를 로드할 때 핵심 요소(예: 비디오, 이미지 등)가 먼저 로드되고, 덜 중요한 요소(예: 댓글 섹션)는 나중에 로드되도록 설정할 수 있습니다.
- 지연 로딩: Angular는 기존에 lazy loading 기능을 제공했지만, 이는 복잡할 수 있었습니다. @defer는 이러한 복잡성을 줄이고, 클라이언트 및 서버 사이드 렌더링 모두에 적합한 통합된 지연 로딩 접근 방식을 제공합니다.

### 장점

- 성능 향상: 핵심 UI 요소가 먼저 로드되므로 페이지의 초기 렌더링 속도가 빨라집니다.
- 자원 절약: 사용자가 실제로 필요한 요소만 로드하여 자원을 절약할 수 있습니다.
- 간편한 구현: 복잡한 lazy loading 설정 없이 간단한 블록 구조로 지연 로딩을 구현할 수 있습니다.


### 사용 방법

@defer는 템플릿 내에서 다음과 같은 블록으로 사용됩니다:

```html
{% raw %}
@defer(condition) {
...deferred stuff
}
{% endraw %}
```

- condition: 이 조건이 충족되면 블록 내의 내용이 로드됩니다. 예를 들어, 특정 버튼 클릭 시 로드될 수 있습니다.
- deferred stuff: 이 블록 내에서 참조되는 컴포넌트, 지시어, 파이프 등이 지연 로드됩니다.

## 지연 로딩의 다양한 단계 관리 방법

### @placeholder

사용자가 나중에 @defer 블록을 통해 로드된 코드로 대체될 초기 콘텐츠를 표시하고 싶을 때가 있습니다.

이를 위해 초기 콘텐츠를 @placeholder 블록으로 감싸면 됩니다:

```html
{% raw %}
@defer {
	<large-component />
} 
@placeholder {
	<initial-content />
}
{% endraw %}
```

defer가 수행되면 placeholder는 제거되고 그 위치에 large-component가 표시됩니다.

#### **@placeholder 매개변수**

@placeholder 블록은 minimum이라는 하나의 선택적 매개변수를 가질 수 있으며, 이는 초 또는 밀리초 단위의 시간 지속 기간입니다.<br/>
minimum은 플레이스홀더 블록이 사용자에게 표시될 최소 시간을 설정하는 데 사용됩니다:

```html
{% raw %}
@defer {
	<large-component />
} 
@placeholder (minimum 2s) {
	<initial-content />
}
{% endraw %}
```

위의 코드는 &lt;initial-content /&gt; 를 2초 동안 렌더링한 후 &lt;large-component /&gt; 가 나타나게 합니다.
네트워크 속도에 따라 @defer 블록이 너무 빨리 표시되어 플레이스홀더를 잠깐 동안만 표시하는데 이 경우 사용자는 페이지에서 이상한 깜박임 효과를 보게 되어 무언가가 고장난 것처럼 느낄 수 있습니다.

### @loading

@loading 블록은 @defer 블록이 백그라운드에서 자바스크립트 번들을 로드하는 동안 일부 콘텐츠를 표시하는 데 사용됩니다.

```html
{% raw %}
@defer {
	<large-component />
} 
@loading {
	<loading-spinner />
}
{% endraw %}
```

로딩이 완료되면 &lt;loading-spinner /&gt; 는 페이지에서 제거되고 &lt;large-component /&gt; 가 그 자리에 렌더링됩니다.

#### @loading 매개변수

minimum과 after를 허용합니다:
- minimum: @loading 블록이 사용자에게 표시될 최소 시간을 지정하는 데 사용됩니다.
- after: 로딩 프로세스가 시작된 후 @loading 표시기를 표시하기 전에 기다려야 하는 시간을 지정하는 데 사용됩니다.

```html
{% raw %}
@defer {
<large-component />
} @loading (after 1s; minimum 2s) {
<loading-spinner />
}
{% endraw %}
```

로딩 표시기는 로딩이 1초 이상 걸리는 경우에만 표시되며, 그렇지 않으면 절대 표시되지 않습니다.<br/>
로딩 표시기가 표시되고 나서 로딩이 완료되면 최소 2초 동안 표시된다는 것을 의미합니다.

#### @placeholder와 @loading의 차이점

두 블록은 유사해 보이지만 다른 목적을 가지고 있습니다.<br/><br/>

@placeholder는 @defer 블록의 내용이 렌더링될 준비가 될 때까지 처음에 표시됩니다.<br/>
이 블록은 번들 로딩이 시작되기 전에도 표시됩니다. 로딩이 사용자가 버튼을 클릭한 후에만 트리거될 수도 있습니다.<br/>
따라서 번들 로딩이 트리거되기 전까지 사용자에게 일부 콘텐츠를 표시하고 싶을 수 있으며, 이때 @placeholder가 사용됩니다.<br/><br/>

반면 @loading 블록은 @defer 블록의 번들 로딩이 시작되고 진행 중일 때만 표시됩니다.<br/>
로딩이 완료되면 사라집니다.

### @error

@error 블록은 @defer 블록의 로딩이 어떤 이유로 실패했을 때 콘텐츠를 표시하는 데 사용됩니다.

```html
{% raw %}
@defer {
	<large-component />
} @error {
	<error-message />
}
{% endraw %}
```

## when
특정 조건이 충족될 때만 컴포넌트를 렌더링하도록 지정하는 역할을 합니다. <br/>
when 뒤에 오는 표현식이 true일 경우에만 해당 컴포넌트가 렌더링됩니다. <br/>
이를 통해 조건부 렌더링을 쉽게 구현할 수 있습니다.

```ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-main',
  template: `
    <button (click)="toggleComments()">댓글 보기</button>
    
    @defer(when showComments) {
      <app-comment-section></app-comment-section>
    }
  `
})
export class MainComponent {
  showComments = signal(false);  // 댓글 섹션의 표시 여부를 관리하는 Signal

  toggleComments() {
    this.showComments.set(!this.showComments());  // 현재 상태를 반전시킵니다.
  }
}

```

## trigger

내장 함수들을 사용해서 조건을 상세하게 작성할 수 있습니다.

| trigger | 설명 |
| --- | --- |
| idle | 브라우저가 유휴 상태에 도달했을 때 지연 로딩을 트리거합니다. |
| interaction | 클릭, 포커스, 터치 및 입력 이벤트가 발생할 때 지연 로딩을 트리거합니다. |
| immediate | 클라이언트가 렌더링을 마친 직후에 지연 로드를 트리거합니다. |
| timer(x) | 정된 시간이 지난 후에 지연 로딩을 트리거합니다. x는 시간(밀리초)입니다. |
| hover | 마우스가 지정된 트리거 영역 위에 호버될 때 지연 로딩을 트리거합니다. |
| viewport | 지정된 콘텐츠가 뷰포트에 들어올 때 지연 블록을 트리거합니다. |

### on

내장 trigger들을 활용하려면 on을 사용해야 합니다.

#### idle (default; defer의 조건을 생략하면 기본 값으로 적용)


```html
{% raw %}
@defer (on idle) {
	<large-cmp />
}
{% endraw %}
```

#### interaction
interaction 트리거는 사용자가 특정 요소와 상호작용할 때 지연된 콘텐츠를 로드하는 기능입니다. 클릭(click)이나 키 입력(keydown) 이벤트가 발생하면 defer를 노출합니다.<br/>

기본적으로 플레이스홀더가 상호작용 요소로 작동합니다. 즉, 플레이스홀더를 클릭하거나 키 입력을 하면 @defer 블록이 트리거되어 지연된 콘텐츠가 로드됩니다. 

```html
{% raw %}
@defer (on interaction) {
  <large-cmp />
} @placeholder {
  <div>플레이스홀더</div> // 클릭 시 trigger
}
{% endraw %}
```

또한, 동일한 템플릿 내에서 템플릿 참조 변수를 사용하여 상호작용할 요소를 지정할 수도 있습니다. 예를 들어:

```html
{% raw %}
<div #greeting>Hello!</div>
@defer (on interaction(greeting)) {
	<greetings-cmp />
}
{% endraw %}

```

#### immediate

이 트리거는 @defer 블록을 즉시 트리거합니다. 어떤 이벤트가 발생할 때까지 기다리지 않습니다.

```html
{% raw %}
@defer (on immediate) {
	<large-component />
}
{% endraw %}
```

이것은 브라우저가 유휴 상태가 될 때까지 기다리지 않고 이 이벤트를 트리거한다는 것을 의미합니다.

#### timer(x)

이 트리거는 타이머 기간이 도달했을 때 발생합니다.<br/>
타이머 값은 밀리초(ms) 또는 초(s)로 설정할 수 있습니다.

```html
{% raw %}
@defer (on timer(5s)) {
	<large-component />
}
{% endraw %}
```

위의 코드에서 @defer 블록은 5초 후에 트리거됩니다.<br/>


#### hover

이 트리거는 사용자가 페이지의 요소 위로 마우스를 올릴 때 발생합니다. hover 이벤트는 mouseenter와 focusin입니다.<br/>
이 이벤트는 @placeholder 블록에 단일 노드만 포함된 경우에만 트리거됩니다.

```html
{% raw %}
@defer (on hover) {
	<large-component />
} @placeholder {
	<loading-spinner />
}
{% endraw %}
```

아래의 예제는 #title 요소가 hover되거나 포커스될 때 @defer 블록이 트리거됩니다.

```html
{% raw %}
<div #title>Title</div>

@defer (on hover(title)) {
	<large-component />
}
{% endraw %}
```

#### viewport


```html
{% raw %}
@defer (on viewport) {
  <large-cmp />
} @placeholder {
	placeholder // 이게 보이면 trigger 됨.
}
{% endraw %}
```

또한, 동일한 템플릿 내에서 템플릿 참조 변수를 지정하여 뷰포트에 들어가는 요소를 감시할 수도 있습니다. 

```html
{% raw %}
<div #greeting>Hello!</div>
@defer (on viewport(greeting)) {
	<greetings-cmp />
}
{% endraw %}
```

위의 코드에서는 페이지를 아래로 스크롤하다가 #greeting이라는 요소가 화면에 나타나면, 그때 @defer 블록이 트리거되어 greetings-cmp가 로드됩니다.

## Prefetching @defer

리소스를 미리 메모리에 로드하여 필요할 때 사용할 수 있도록 하는 과정입니다.

```html
{% raw %}
@defer (on timer(5s)) {
	<large-component />
}
{% endraw %}
```

기능적으로는 다음과 동일합니다:

```html
{% raw %}
@defer (on timer(5s); prefetch on idle) {
	<large-component />
}
{% endraw %}
```

번들은 브라우저가 유휴 상태가 될 때 사전 로드되며, 이는 기본 동작입니다.<br/>
그러나 &lt;large-component /&gt; 는 사전 로드가 오래 전에 완료되었더라도 5초 후에만 사용자에게 표시됩니다.<br/><br/>

유휴 트리거는 사전 로드에 대한 훌륭한 기본값이지만, 배운 다른 내장 트리거를 사용할 수도 있고 사용자 정의 트리거를 정의할 수도 있습니다.<br/>
사전 로드 옵션으로 할 수 있는 몇 가지 예를 살펴보겠습니다.

### 뷰포트에서 사전 로드하고 상호작용 시 표시

페이지 아래에 큰 컴포넌트가 있어 사용자가 페이지를 아래로 스크롤할 때만 로드하고 싶다고 상상해보세요.<br/>
그런 다음 사용자가 입력 상자에 무언가를 입력할 때만 표시하고 싶습니다.<br/>
다음과 같이 할 수 있습니다:

```html
{% raw %}
@defer (on interaction; prefetch on viewport) {
	<large-component />
} @placeholder {
	<input />
}
{% endraw %}
```

모든 내장 트리거는 사전 로드와 @defer 블록 표시 모두에 사용할 수 있습니다.

### when 키워드를 사용한 사용자 정의 @defer 트리거

필요한 경우 when 키워드를 사용하여 사용자 정의 트리거를 정의할 수 있습니다.

```ts
@Component({
	selector: "app",
	template: `
		<button (click)="onLoad()">사전 로드 트리거</button>
		<button (click)="onDisplay()">표시 트리거</button>

		@defer(when show; prefetch when load) {
		<large-component />
		}
	`,
})
export class AppComponent {
	load: boolean = false;
	show: boolean = false;

	onLoad() {
		this.load = true;
	}

	onDisplay() {
		this.show = true;
	}
}
```

이 예에서는 사전 로드 및 표시 이벤트가 사용자 정의 트리거를 사용하여 구성됩니다.<br/><br/>

사전 로드 버튼을 클릭하면 @defer 자바스크립트 번들의 로딩이 백그라운드에서 트리거됩니다.<br/>
그러나 show 변수가 여전히 false이기 때문에 @defer 블록은 아직 표시되지 않습니다.<br/><br/>

그런 다음 표시 버튼을 클릭하면 번들 로딩이 완료된 후 @defer 블록이 표시됩니다.<br/>
반면, 페이지를 새로 고치고 이번에는 먼저 표시 버튼을 클릭합니다.<br/>
이 경우 사전 로드가 트리거되고 블록이 즉시 표시되는 것을 알 수 있습니다.<br/><br/>

이는 사용자 정의 사전 로드 트리거를 구성할 수 있지만, 표시 트리거가 먼저 실행되면 사전 로드가 즉시 수행되고 사전 로드 조건이 무시된다는 것을 의미합니다.<br/>
이는 번들이 먼저 로드되지 않으면 @defer 블록을 표시할 수 없기 때문에 합리적입니다.

### @defer와 @if의 차이

@defer를 사용하면 논리적 조건이 트리거되고 구성 요소가 로드되어 표시되면 다시 숨길 수 없습니다. <br/>
@if는 조건에 따라 다시 숨길 수 있습니다.<br/><br/>

둘을 조합하여 표현할 수도 있습니다.

```html
{% raw %}
@defer (on interaction; prefetch on viewport) { 
	@if (someCondition) {
		<large-component />
	} 
	@placeholder {
		<placeholder-component />
	}
}
{% endraw %}
```

이 설정을 통해 &lt;large-component /&gt; 는 사용자가 페이지를 아래로 스크롤할 때만 로드됩니다.<br/><br/>

@defer 블록이 페이지에 적용되지만, 이는 &lt;large-component /&gt;가 아직 표시된다는 의미는 아닙니다.<br/>
&lt;large-component /&gt;는 someCondition이 true인 경우에만 표시되며, 조건이 false인 경우 숨겨진 상태로 유지됩니다.<br/><br/>

결론적으로 @defer는 코드가 로드되고 구성 요소에 적용되는 시점을 제어하기 위한 것이며, @if는 구성 요소의 가시성을 제어하기 위한 것입니다.