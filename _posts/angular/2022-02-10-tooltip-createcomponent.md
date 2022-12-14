---
title: "createComponent로 툴팁 만들기 (Create Tooltip with createComponent)"
date: 2022-02-10 17:46:00 +0900
comments: true
categories: angular
tags: [dynamic, component, createcomponent, tooltip]
---

dynamic component 기능인 `createComponent`를 활용하여 Tooltip을 만들어 봅시다.<br/>
<br/>
# 기본 동작

div 2개와 tooltip 2개를 만들어 각각을 `mouseenter`, `mouseleave` 했을 때 서로 다른 툴팁이 표시되었다가 제거되는 코드를 작성합니다.<br/>

1. `tooltip component`를 2개 생성합니다.
2. `tooltip component`를 호출할 `container component`를 생성합니다.
3. `createComponent`를 통해 `tooltip component`를 호출할 `directive`를 생성합니다.

<br/>

# component 생성하기


## container component

{% raw %}<div>{% end raw %} 2개를 만들고 `directive`를 호출하는 형태로 구성합니다.

```tsx
@Component({
  selector: 'app-container',
  template: `
  <div [tooltip]="1">Show A-Tooltip</div>
  <div [tooltip]="2">Show B-Tooltip</div>
  `
})
export class ContainerComponent {}
```

<br/>

## dynamic components

호출할 두개의 tooltip component를 작성해봅시다.

```tsx
@Component({
  selector: 'a-tooltip',
  template: `
    <p>Hello World</p>`
})
export class ATooltipComponent {}

@Component({
  selector: 'b-tooltip',
  template: `
    <p>Good bye</p>`
})
export class BTooltipComponent {}
```

<br/>

# directive 생성하기

directive에 이벤트를 정의하고 타입에 따라 각기 다른 Tooltip을 연결해 줍니다. <br/>

## tooltip directive

```tsx
@Directive({ selector: '[tooltip]' })
export class TooltipDirective {
  @Input() tooltip!: number;

  componentRef!: ComponentRef<any>;

  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver // do not need if angular version is 13 or above
  ) {

  }

  @HostListener('mouseenter')
  load() {
    if(!this.tooltip) return;

    const componentInfo = this.getComponentInfo();

    // under Angular 13
    const componentRef = this.resolver.resolveComponentFactory(componentInfo?.component as Type<unknown>);
    this.componentRef = this.viewContainerRef.createComponent<any>(componentRef);

    // Angular 13 or above, createComponent with componentInfo directly.
    // this.componentRef = this.viewContainerRef.createComponent<any>(componentInfo);
    
  }

  @HostListener('mouseleave', ['$event'])
  unload(event: any) {
    componentRef?.destroy();
  }

  private getComponentInfo(type?: number) {
    let component;

    switch(type) {
      case 1:
        component = ATooltipComponent;
        break;
      case 2:
        component = BTooltipComponent;
        break;
      default:
        return;
    }
    return component;
  }
}
```

실행해서 마우스를 움직여보면 올바르게 동작하는 것을 확인할 수 있습니다.<br/>


# 주의사항

`module`에 `directive`와 `TooltipComponent` 들이 모두 정의되어야 합니다. <br/>
특히 위 예제의 경우 `directive`가 `component` 정보를 담고 있으므로 한 `module`에 모두 정의 되어야 합니다.

