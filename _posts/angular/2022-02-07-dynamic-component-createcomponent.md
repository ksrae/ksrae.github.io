---
title: "createComponent으로 동적 컴포넌트 만들기 (Dynamic Component with createComponent)"
date: 2022-02-07 17:46:00 +0900
comments: true
categories: angular
tags: [dynamic, component, createcomponent]
---

보다 범용적인 버전에서 활용할 수 있는 createComponent를 사용하여 dynamic component를 구현합니다.<br/>

# 기본 원리

1. componentFactory를 활용하여 호출할 component의 정보를 가져옵니다.
2. component를 담을 container component의 viewContainerRef를 가져옵니다.
3. viewContainerRef의 createComponent함수를 통해 호출할 component를 container component에 랜더링 합니다.
4. viewContainerRef의 clear 함수를 활용하여 호출한 component를 제거 합니다.


# component 생성하기

## container component

dynamic component를 호출하는 container에 해당하는 component를 작성해 봅시다.<br/>
버튼 2개를 클릭했을 때 서로 다른 component가 화면에 출력되는 예제 입니다.

```tsx
@Component({
  selector: 'app-container',
  template: `
  <button (click)="callAComponent()">Show A-Component</button>
  <button (click)="callBComponent()">Show B-Component</button>
  `
})
export class ContainerComponent {
  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver
  ) {}

  callAComponent() {
    const resolve = this.resolver.resolveComponentFactory(AComponent);
    this.viewContainerRef.crateComponent(resolve);
  } 
  callBComponent() {
    const resolve = this.resolver.resolveComponentFactory(BComponent);
    this.viewContainerRef.crateComponent(resolve);
  }
}
```

## dynamic components

호출할 두개의 component를 작성해봅시다. css를 작성하면 보다 확실히 확인할 수 있으나 여기에서는 생략합니다.

```tsx
@Component({
  selector: 'a-component',
  template: `
    <p>Hello World</p>`
})
export class AComponent {

}

@Component({
  selector: 'b-component',
  template: `
    <p>Good bye</p>`
})
export class BComponent {

}
```

예제를 실행하면 버튼을 클릭하여 AComponent, BComponent를 스위치하여 화면에 표시합니다.


## 값을 주입하는 방법

값을 주입하는 방법은 매우 간단합니다. createComponent의 리턴 값을 받아 instance에 값을 주입하면 됩니다.<br/>
여기에서 instance란 쉽게 말해 호출할 component의 public 변수 / 함수 라고 생각하면 됩니다.<br/>
<br/>
또한, @Input() 으로 선언된 변수에도 주입이 가능하지만 @Input('') set ... 에는 적용되지 않습니다.<br/>
그리고, onchange 사이클에도 잡히지 않으므로 주의하여야 합니다.<br/>
<br/>
위의 component들을 다시 작성해보겠습니다.<br/>


### container component

dynamic component를 호출하는 container에 해당하는 component를 작성해 봅시다.<br/>
버튼 2개를 클릭했을 때 서로 다른 component가 화면에 출력되는 예제 입니다.

```tsx
@Component({
  selector: 'app-container',
  template: `
  <button (click)="callAComponent()">Show A-Component</button>
  <button (click)="callBComponent()">Show B-Component</button>
  `
})
export class ContainerComponent {
  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver
  ) {}

  callAComponent() {
    const resolve = this.resolver.resolveComponentFactory(AComponent);
    const componentRef = this.viewContainerRef.crateComponent(resolve);
    componentRef.instance.data = 'hello world';
  }
  callBComponent() {
    const resolve = this.resolver.resolveComponentFactory(BComponent);
    const componentRef = this.viewContainerRef.crateComponent(resolve);
    componentRef.instance.data = 'good bye';
  }
}
```

### dynamic components

```tsx
@Component({
  selector: 'a-component',
  template: `
    <p>{{data}}</p>`
})
export class AComponent {
  data!: string;
}

@Component({
  selector: 'b-component',
  template: `
    <p>{{data}}</p>`
})
export class BComponent {
  @Input() data!: string;
}
```

실행해보면 data 변수를 통해 주입된 값이 표시됨을 확인할 수 있습니다.<br/>
BComponent의 @Input()은 이런 형태도 잘 주입 된다는 것을 테스트 해보기 위해 작성한 것이며, 의미는 없습니다.


## 값 주입이 안되는 경우

만일 값을 2회 이상 주입한다면 instance 통해 주입하더라도 값이 변경되지 않습니다.<br/>
afterViewInit에서 확인해보면 값이 들어오고 있음을 확인할 수 있는데 이 때 changeDetection을 통해 랜더링 시켜주어야 비로소 화면에 적용됩니다.

### container component
```tsx
...
export class ContainerComponent implements AfterViewInit {
  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver,
    private cd: ChangeDetectionRef
  ) {}
  ngAfterViewInit() {
    this.cd.markForCheck();
  }
```

## Subject를 활용한 값 주입

changeDetection 사용이 꺼려진다면 Subject를 활용하는 것도 좋은 방안이 될 수 있습니다.<br/>
초기값을 가질 수 있는 BahaviorSubject를 활용한다면 보다 쉽게 값을 주입할 수 있습니다.


### container component

```tsx
@Component({
  selector: 'app-container',
  template: `
  <button (click)="callAComponent()">Show A-Component</button>
  <button (click)="callBComponent()">Show B-Component</button>
  `
})
export class ContainerComponent {
  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver
  ) {}

  callAComponent() {
    const resolve = this.resolver.resolveComponentFactory(AComponent);
    const componentRef = this.viewContainerRef.crateComponent(resolve);
    componentRef.instance.data$.next('hello world');
  }
  callBComponent() {
    const resolve = this.resolver.resolveComponentFactory(BComponent);
    const componentRef = this.viewContainerRef.crateComponent(resolve);
    componentRef.instance.data.next('good bye');
  }
}
```

### dynamic components

```tsx
@Component({
  selector: 'a-component',
  template: `
    <p>{{data$ | async}}</p>`
})
export class AComponent {
  data$ = new BehaviorSubject<any>('');
}

@Component({
  selector: 'b-component',
  template: `
    <p>{{data$ | async}}</p>`
})
export class BComponent {
  data$ = new BehaviorSubject<any>('');
}
```


# Component 제거하기

제거는 매우 간단하게 container component의 viewContainerRef의 clear 함수를 호출하면 됩니다.

### container component

```tsx
@Component({
  selector: 'app-container',
  template: `
  <button (click)="callAComponent()">Show A-Component</button>
  <button (click)="callBComponent()">Show B-Component</button>
  <button (click)="removeAll()">RemoveAll</button>
  `
})
export class ContainerComponent {
  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver
  ) {}

  callAComponent() {
    const resolve = this.resolver.resolveComponentFactory(AComponent);
    const componentRef = this.viewContainerRef.crateComponent(resolve);
    componentRef.instance.data$.next('hello world');
  }
  callBComponent() {
    const resolve = this.resolver.resolveComponentFactory(BComponent);
    const componentRef = this.viewContainerRef.crateComponent(resolve);
    componentRef.instance.data.next('good bye');
  }
  removeAll() {
    this.viewContainerRef.clear();
  }
}
```


# 기타
## v13 이후 개선사항

v13부터는 기존의 createComponent 함수가 deprecated 되고, 새로 작성된 createComponent 함수를 사용해야 합니다.<br/>
기존에는 componentFactory를 통해 component에 접근해야 했는데 새로운 버전에서는 component에 직접 접근할 수 있게 되어 더욱 간결한 코딩이 가능해졌습니다.<br/>
<br/>
위의 container component를 v13 버전으로 작성하면 다음과 같습니다.

### container component

```tsx
@Component({
  selector: 'app-container',
  template: `
  <button (click)="callAComponent()">Show A-Component</button>
  <button (click)="callBComponent()">Show B-Component</button>
  `
})
export class ContainerComponent {
  constructor(
    private viewContainerRef: ViewContainerRef
  ) {}

  callAComponent() {
    const componentRef = this.viewContainerRef.crateComponent(AComponent);
    componentRef.instance.data$.next('hello world');
  }
  callBComponent() {
    const componentRef = this.viewContainerRef.crateComponent(BComponent);
    componentRef.instance.data.next('good bye');
  }
}
```


# 참고 사이트
- [Angular 공식 사이트 dynamic-component-loader](https://angular.io/guide/dynamic-component-loader)