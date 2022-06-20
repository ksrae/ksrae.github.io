---
title: "createComponent로 모달 만들기 - 기본 (Create Modal with createComponent)"
date: 2022-02-15 00:00:00 +0900
comments: true
categories: angular
tags: [dynamic, component, createcomponent, modal]
---

dynamic component 기능인 createComponent를 활용하여 Modal 만들어 봅시다.<br/>
<br/>


# modal의 정의

tooltip과는 다르게 modal은 고려해야할 사항이 여러가지가 있습니다. 그중에서 기본적으로 modal이 갖추어야할 기능들을 추려서 정리해보고 구현해보려고 합니다.<br/>
1. modal은 이벤트 이후 조건에 의해 호출되는 경우가 많고, 그 조건이 다양하므로 tooltip이 directive를 사용하여 이벤트에 의해 즉시 호출되는 것과는 다른 방식으로 구현해야 합니다.
2. modal은 modal위에 modal이 겹쳐서 나올 케이스가 있으므로 이를 고려해야 합니다.
3. modal은 리턴값을 받아야 합니다. 즉, 호출한 modal의 결과를 기다렸다가 처리하는 코드를 작성하도록 고려해야 합니다.



# 기본 동작

버튼 2개를 눌러 2개의 모달을 띄우는 코드를 작성합니다.<br/>

1. component를 호출할 controller를 생성합니다.
  - 호출할 modal component을 create하고 return 값을 처리할 result$를 주입합니다.
  - result$를 통해 전달 받은 값을 호출한 container component에 return 합니다.
2. modal component를 2개 생성합니다.
  - 버튼 이벤트에 따라 주입된 result$에 값을 입력합니다.
3. modal component를 호출할 container component를 생성하고 버튼 2개를 만듭니다.
  - controller를 통해 modal을 생성하고, controller의 open 함수를 통해 모달을 호출합니다.
  - 결과를 Observable 변수에 연결하고, template에 직접 결과를 표시하도록 합니다.




<br/>

# controller 생성하기

Tooltip에서 directive의 역할과 유사한 역할을 담당합니다.<br/>
이벤트를 정의하고 타입에 따라 각기 다른 modal을 연결해 주는 역할을 합니다. <br/>

## modal container

```tsx
export class ModalController {
  result$: Subject<any> = new Subject<any>();
  componentRef!: any;
  viewRef!: ViewContainerRef;
  resolver!: ComponentFactoryResolver;

  constructor(
    private viewContainerRef: ViewContainerRef,
    private factoryResolver: ComponentFactoryResolver
  ) {
    this.viewRef = viewContainerRef;
    this.resolver = factoryResolver;
  }

  open(type: number): Observable<any> {
    const componentInfo = this.getComponentInfo(type);
    // v12 이하 버전에서만 적용합니다.
    const componentFactory = this.resolver.resolveComponentFactory(componentInfo as Type<unknown>)
    // v13인 경우 factory가 아닌 component를 주입할 수 있습니다.
    this.componentRef = this._viewContainerRef.createComponent<any>(componentFactory);
    this.componentRef.instance.result$ = this.result$;

    // return observable return value then close.
    return this.result$.pipe(
      first(),
      finalize(() => this.close())
    );
  }

  close() {
    this.componentRef.destroy();
  }

  private getComponentInfo(type: number) {
    let component;

    switch(this.type) {
      case 1:
        component = AModalComponent;
        break;
      case 2: 
        component = BModalComponent;
        break;
      default:
        return;
    }

    return component;
  }
}
```

# component 생성하기

<br/>

## container component

button 2개를 만들고 모달을 호출하는 형태로 구성합니다.

```tsx
@Component({
  selector: 'app-container',
  template: `
    <button (click)="openModal(1)">Show Modal 1</button>
    <button (click)="openModal(2)">Show Modal 2</button>
    <p>{{modalResult$ | async}}</p>
  `
})
export class ContainerComponent {
  modalResult$ = Observable<any>();

  constructor(
    private viewContainerRef: ViewContainerRef,
    private resolver: ComponentFactoryResolver
  ) {}

  openModal(type: number) {
    const modal = new ModalController(this.viewContainerRef, this.resolver);
    this.modalResult$ = modal.open(type).pipe(first());
  }
}
```

<br/>

## dynamic components

호출할 두개의 modal component를 작성해봅시다.

```tsx
@Component({
  selector: 'a-modal',
  template: `
    <div class="modal">
      <p>modal-a works!</p>
      <button (click)="cancel()">취소</button>
      <button (click)="confirm()">확인</button>
    </div>
    `
})
export class AModalComponent {
  @Input() result$!: Subject<boolean>;
  cancel() {
    this.result$.next(false);
  }
  confirm() {
    this.result$.next(true);
  }
}

@Component({
  selector: 'b-modal',
  template: `
    <div class="modal">
      <p>modal-b works!</p>
      <button (click)="cancel()">취소</button>
      <button (click)="confirm()">확인</button>
    </div>`
})
export class BModalComponent {
  @Input() result$!: Subject<boolean>;
  cancel() {
    this.result$.next(false);
  }
  confirm() {
    this.result$.next(true);
  }  
}
```

실행해서 버튼을 클릭하면 올바르게 동작하는 것을 확인할 수 있습니다.<br/>
단, 이 예제에서는 중복 모달을 처리하지 않았으므로 같은 모달이 여러개 표시될 수 있습니다.<br/>

# 결과

기본적인 모달의 형태를 구성할 수 있는 코드를 작성하였지만 아직 실제로 쓰기에는 많이 부족한 면이 있습니다.<br/>
예를 들어, 같은 모달이 중복되어 노출되거나 모달을 직접 닫는다거나 하는 기능이 없고, 모달이 겹칠 때 호출한 순서에 따른 z-index를 변경하는 등의 요소들이 추가 개발되어야 합니다. <br/>
<br/>
그럼에도 다중 모달을 띄우고 관리할 수 있다는 점에서 큰 장점이 있어서 매우 효율적인 방법이라고 생각합니다.<br/>
<br/>
위에서 언급한 남은 과제들은 다른 글에서 다루어 보기로 하고 글을 마칩니다.<br/>
