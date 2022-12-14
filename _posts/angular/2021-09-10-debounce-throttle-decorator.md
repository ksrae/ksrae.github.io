---
title: "Debounce와 Throttle의 Directive와 Decorator 만들기 (How to create decounce and throttle directives and decorators?)"
date: 2021-09-10 17:54:00 +0900
comments: true
categories: javascript
tags: [decorator, directive, debounce, throttle, rxjs, typescript]
---

Debounce 또는 Throttle Decorator를 만들기 위해 여러 사이트를 검색하였으나, Angular나 Typescript에 적합한 예제를 찾을 수 없어서 직접 만들었습니다.<br/><br/>
저와 같이 검색하시는 분들께 도움이 되었으면 좋겠다는 생각을 하며 글을 시작합니다.<br/><br/>

[decorator에 대해서는 이전 글에 자세히 기록하였으니 참고](https://ksrae.github.io/javascript/decorator/)하여 주시고,<br/>
여기에서는 debounce와 throttle에 대해서 알아보고, directive와 decorator를 각각 만들어보고 비교하는 시간을 가지려고 합니다. <br/><br/>

먼저 debounce에 대해서 알아봅시다.

## debounce의 정의
debounce는 요청이 들어온 후 일정 시간 뒤에 요청이 수행되는 것을 말합니다.<br/>
단, 대기 시간 중에 다른 요청이 들어올 경우 이전 요청은 취소되며 새로운 요청이 들어온 시점부터 다시 일정 시간 뒤에 요청이 수행됩니다.<br/><br/>

이를 directive와 decorator로 구현하면 아래와 같습니다.

### debounce directive 만들기
directive는 rxjs를 활용하여 간단히 작성할 수 있습니다.

```tsx
@Directive({
	selector: '[debounce]'
})
export class DebounceDirective {
  @Output() debounceClick = new EventEmitter();

  constructor(private viewContainer: ViewContainerRef) {

  }

  ngOnInit() {
    fromEvent<MouseEvent>(this.viewContainer.element.nativeElement, 'click').pipe(debounceTime(500)).subscribe((e) => {
      this.debounceClick.emit(e);
    });
  }
}
```

debounce Directive를 활용하기 위해 Template을 다음과 같이 작성해야 합니다.

```html
{% raw %}
<button debounce (debounceClick)="submit($event)">
{% endraw %}
```

* 위의 코드는 debounceTime을 고정한 경우이며, debounceTime을 파라미터로 받을 경우 일부 수정이 필요합니다.<br/>


### debounce decorator 만들기

debounce를 decorator로 만들어 보겠습니다. 원리는 다음과 같습니다.<br/>
이벤트가 들어오면 이전 이벤트를 취소하고, 새로운 이벤트를 ms시간 후에 실행하는 것이며, 이를 구현하면 다음과 같습니다.


```tsx
export function Debounce(ms: number = 0) {
  return (target : any, propertyKey : string, descriptor: PropertyDescriptor) => {
    const callback = descriptor.value;
    let runTime!: any;

    descriptor.value = (prop: unknown) => {
      if(runTime) {
          clearTimeout(runTime);
      }

      runTime = setTimeout(() => {
          callback(prop);
      }, ms);
    }
  }
}
```

이를 활용하려면 component를 다음과 같이 작성해야 합니다.

```tsx
export class AppComponent {
	...
	@Debounce(1000)
	submit(e: Event) {
		...
	}
}
```


## throttle의 정의
throttle의 정의는 일정 시간 동안 요청이 한번만 수행되는 것 입니다.<br/>
즉, `throttleTime`이 1000ms이라고 하면, 첫번째 요청이 실행되고 1000ms 동안은 어떠한 요청이 와도 무시되며, `throttleTime`이 종료된 이후에 요청을 새로 받아 수행합니다.


### throttle의 directive 만들기
throttle도 debounce와 마찬가지로 rxjs를 활용해 간단히 작성할 수 있습니다.


```tsx
@Directive({
	selector: '[throttle]'
})
export class ThrottleDirective {
  @Output() throttleClick = new EventEmitter();

  constructor(private viewContainer: ViewContainerRef) {

  }

  ngOnInit() {
    fromEvent<MouseEvent>(this.viewContainer.element.nativeElement, 'click').pipe(throttleTime(500)).subscribe((e) => {
      this.throttleClick.emit(e);
    });
  }
}
```

throttle Directive를 활용하기 위해 Template를 다음과 같이 작성해야 합니다.

```html
{% raw %}
<button throttle (throttleClick)="submit($event)">
{% endraw %}
```


### throttle decorator 만들기

```tsx
export function Throttle(ms: number = 0) {
  return (target : any, propertyKey : string, descriptor: PropertyDescriptor) => {
    const callback = descriptor.value;
    let runTime!: any;
    let isrunning: boolean = false;

    descriptor.value = (prop: unknown) => {
      if(!isrunning) {
        callback(prop);
        clearTimeout(runTime);
        isrunning = true;
        runTime = setTimeout(() => {
            isrunning = false;
        }, ms);
      } 
    }
  }
}
```

이를 활용하려면 component를 다음과 같이 작성해야 합니다.

```tsx
export class AppComponent {
	...
	@Throttle(1000)
	submit(e: Event) {
		...
	}
}
```

## debounce / throttle directive와 decorator의 차이
directive로 작성할 경우 코드를 간결하게 작성할 수 있고, Angular 코드로 이므로 관리가 쉬운 반면, Template에서 클릭 이벤트를 작성 시에 일반적인 클릭 이벤트가 아닌 directive 호출 및 directive에 설정된 Emit 이벤트를 매번 입력해야 하므로 불편하다는 단점이 있습니다.<br/>
반면에 decorator로 작성할 경우 ts코드로 작성하므로 코드가 길어지고 Angular 코드가 아니므로 관리가 어려운 면이 있으나, Template에서 기존 명령을 사용하므로 간결해지고, click 이벤트를 처리하는 component 함수에서 직접 time을 조절할 수 있으므로 직관적입니다. <br/><br/>

두 케이스 중 어느 것이 좋다고 하기 보다는 개발자의 취향에 따라 선택하는 것이 좋다고 생각합니다.




