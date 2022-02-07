---
title: "Dynamic Component with ngComponentOutlet"
date: 2021-11-21 23:31:00 +0900
comments: true
categories: angular
tags: [dynamic, component, ngcomponentoutlet]
---

# ngIf를 활용하여 조건에 맞는 component만 노출하기

일반적으로 component를 조건에 따라 교체하는 것은 불가능합니다.<br/>
따라서 여러 component를 template에 나열하고, 이를 ngIf로 원하는 component만 노출하고, 나머지는 감추는 방법을 사용했습니다.<br/>
이를 코드로 작성하면 아래와 같습니다.

```html
<component-a *ngIf="type==TYPEA"></component-a>
<component-b *ngIf="type==TYPEB"></component-b>
<component-c *ngIf="type==TYPEC"></component-c>
<component-d *ngIf="type==TYPED"></component-d>
```

이러한 방식은 html에서 직관적으로 조건에 맞는 component를 코드상으로 확인할 수 있는 장점이 있으나 랜더링에 부하를 줄 수 있을 뿐 아니라 html 코드가 지저분해진다는 단점이 있습니다.

# ngComponentOutlet

ngComponentOutlet은 component코드 상에서 template에 원하는 component만 골라 랜더링 할 수 있도록 해줍니다.<br/>

사용방법은 template에 component가 표시될 dom에 ngComponentOutlet을 선언하고 component명을 입력하는 것으로 간단하게 사용할 수 있습니다.

```tsx
@Component({
  selector: 'app-root'
  template: `
    <ng-container *ngComponentOutlet="component"></ng-container>
  `
})
export class AppComponent {
  component = AComponent;
}
```
위의 코드에서 ng-container 에 AComponent가 적용되어 표시됨을 쉽게 확인할 수 있습니다.


## 여러 component 중 선택하기

ngIf와는 다르게 ngComponentOutlet은 component 코드 내에서 원하는 component를 선택할 수 있습니다.

```tsx
@Component({
  selector: 'app-root',
  template: `
    <ng-container *ngComponentOutlet="component"></ng-container>
		<button (click)="change()">change component</button>`
})
export class AppComponent {
  component = AComponent;
	swap = false;

	change() {
		swap = !swap;
		component = swap ? AComponent : BComponent;
	}
}


@Component({
  selector: 'a-component',
  template: `
    <p>Hello World</p>`
})
export class AComponent implements OnInit {

}

@Component({
  selector: 'b-component',
  template: `
    <p>Good bye</p>`
})
export class BComponent implements OnInit {

}
```

예제를 실행하면 버튼을 클릭하여 AComponent, BComponent를 스위치하여 화면에 표시합니다.

# 값을 주입하는 방법

ngComponentOutlet가 ngIf와 다른 큰 특징은 바로 하위 component가 어떤 것이든 관계 없이 같은 값을 주입할 수 있다는 것입니다.<br/>
값을 주입하는 방법은 Injector를 통하는 방법과 content를 활용하는 방법 두가지가 있습니다. 둘 다 동시에 적용할 수도 있습니다.

## Injector 활용


### useValue

```tsx
export const TITLE = new InjectionToken<string>('app.title');

// parent component
@Component({
  selector: 'app-root',
  template: `
    <ng-container *ngComponentOutlet="CompleteComponent;
                                      injector: myInjector"></ng-container>`
})
export class AppComponent {
  // This field is necessary to expose CompleteComponent to the template.
  CompleteComponent = CompleteComponent;
  myInjector: Injector;

  constructor(injector: Injector) {
    this.myInjector = Injector.create({providers: [{provide: TITLE, useValue: 'hello world'}],  parent: injector});
  }
}

// child component
@Component({
  selector: 'complete-component',
  template: `Complete: {{titleInjected}}`
})
export class CompleteComponent {
  constructor(@Inject(TITLE) public titleInjected: string) {}
}

```

먼저 component 코드를 살펴보면, myInjector를 Injector형으로 생성하고, 여기에 원하는 값을 provider를 통해 생성합니다.<br/> 
이 때 주의할 점은 parent를 injector로 선언해야 한다는 것입니다.<br/><br/>

template 코드에서는 간단하게 injector 값에 component에서 선언했던 myInjector를 연결하면 됩니다.<br/>
또한, child component에서는 provide로 받은 값을 constructor에서 @Inject()로 받아와 원하는 변수명에 담는 것으로 parent component로부터 전달받은 값을 활용할 수 있습니다.<br/><br/>


이 방법은 일반적으로 parent에서 child로 값을 주입하기 위해 사용하는 @Input()을 사용하지 않으므로 불편할 수 있습니다.<br/>
그러나 이 방법의 장점은 provider를 통하기 때문에 Class나 Factory를 전달 할 수 있다는 점입니다.


### Class

```tsx
export class Greeter {
  suffix = '!';
}

// parent
@Component({
  selector: 'app-root',
  template: `
    <ng-container *ngComponentOutlet="CompleteComponent;
                                      injector: myInjector"></ng-container>`
})
export class AppComponent {
  // This field is necessary to expose CompleteComponent to the template.
  CompleteComponent = CompleteComponent;
  myInjector: Injector;

  constructor(injector: Injector) {
    this.myInjector = Injector.create({providers: [{provide: Greeter, deps: []}], parent: injector});
  }
}

// child
@Component({
  selector: 'complete-component',
  template: `Complete: {{ greeter.suffix }}`
})
export class CompleteComponent {
  constructor(public greeter: Greeter) {}
}
```

이 예제는 Greeter라는 간단한 class를 생성하고 이를 injector를 통해 child로 전달하는 방법을 보여주고 있습니다.<br/>
기본적인 방법은 위의 useValue와 같으므로 설명을 생략하겠습니다.



## Content 활용

이제 다른 방법으로 주입할 수 있는 방법을 알아 보겠습니다.<br/>
바로 content라는 옵션을 활용하는 방법인데 이 방식은 특이하게도 document의 node를 주입할 수 있습니다.<br/>
즉, Injector가 값을 주입하는 역할이라면 content는 template에 node를 주입하는 역할이라고 할 수 있습니다.<br/>
쉽게 말하자면 content를 통해 child에게 특정 dom을 생성하여 전달하고 이를 제어할 수 있습니다.<br/><br/>

먼저 간단하게 text node를 주입하는 방법을 알아보고, 그 다음 dom을 주입하는 방법을 알아보겠습니다.

### textNode
```tsx

// parent
@Component({
  selector: 'app-root',
  template: `
    <ng-container *ngComponentOutlet="CompleteComponent;
                                      content: myContent"></ng-container>`
})
export class AppComponent {
  // This field is necessary to expose CompleteComponent to the template.
  CompleteComponent = CompleteComponent;

	// 반드시 이차원배열의 형태를 유지해야 함.
	// ng-content를 사용하기 위해서는 document.create을 활용해야 함.
  myContent = [[document.createTextNode('Ahoj')], [document.createTextNode('Svet')]];

  constructor() {

  }
}

// child
@Component({
  selector: 'complete-component',
  template: `Complete: <ng-content></ng-content> <ng-content></ng-content>`
})
export class CompleteComponent {
  constructor() {}
}
```
위의 코드를 분석해보면 myContent라는 변수를 이차원 변수로 작성하고, textCode 2개를 각각 배열의 형태로 담고 있습니다.<br/>
content는 반드시 이차원 배열의 형태여야 하며, 각각의 값은 배열의 형태를 유지해야 함을 주의하여야 합니다.<br/><br/>

실행해보면 parent에서 주입한 content는 child의 ng-content 에 각각 적용 주입됨을 확인할 수 있습니다.


### dom
```tsx

// parent
@Component({
  selector: 'app-root',
  template: `
    <ng-container *ngComponentOutlet="CompleteComponent;
                                      content: myContent"></ng-container>`
})
export class AppComponent {
  CompleteComponent = CompleteComponent;
  button = document.createElement('button');
  myContent = [[this.button]];

  constructor() {
    this.button.textContent = 'Click me';
    this.button.onclick = () => {
      console.log('click');
    };
  }
}

// child
@Component({
  selector: 'complete-component',
  template: `Complete: <ng-content></ng-content>`
})
export class CompleteComponent {
  constructor() {}
}
```

위의 예제는 버튼을 주입하고, 그 이벤트를 부모가 처리하는 방법 입니다.<br/>
button의 attribute 및 이벤트를 모두 설정하여 전달할 수 있습니다.<br/>
기본적인 방식은 textNode와 같으므로 생략합니다.


## Injector와 content 모두 활용하기
```tsx
export class Greeter {
  suffix = '!';
}

export const TITLE = new InjectionToken<string>('app.title');

// parent
@Component({
  selector: 'app-root',
  template: `
    <ng-container *ngComponentOutlet="CompleteComponent;
                                      injector: myInjector;
                                      content: myContent"></ng-container>`
})
export class AppComponent {
  // This field is necessary to expose CompleteComponent to the template.
  CompleteComponent = CompleteComponent;
  myInjector: Injector;

  myContent = [[document.createTextNode('Ahoj')], [document.createTextNode('Svet')]];

  constructor(injector: Injector) {
    this.myInjector = Injector.create({providers: [{provide: TITLE, useValue: 'hello world'}],  parent: injector});
  }
}

// child
@Component({
  selector: 'complete-component',
  template: `Complete: <ng-content></ng-content> <ng-content></ng-content>{{ titleInjected }}`
})
export class CompleteComponent {
  constructor(@Inject(TITLE) public titleInjected: string) {}
}
```

설명은 위의 예제들을 참고하시기 바랍니다.


# 연결된 component 제거하기
ngComponentOutlet에 연결되어 있는 component를 변경하는 것 외에 제거도 가능합니다.<br/>
쉽게 component 값에 null을 대신 전달하면 되며 예제는 다음과 같습니다.

```tsx
@Component({
  selector: 'app-root',
  template: `
    <ng-container *ngComponentOutlet="component"></ng-container>
		<button (click)="change()">change component</button>`
})
export class AppComponent {
  component = AComponent;
	swap = false;

	change() {
		swap = !swap;
		component = swap ? AComponent : null;
	}
}


@Component({
  selector: 'a-component',
  template: `
    <p>Hello World</p>`
})
export class AComponent implements OnInit {

}
```

# 참고 사이트
- [Angular 공식 사이트](https://angular.io/api/common/NgComponentOutlet)