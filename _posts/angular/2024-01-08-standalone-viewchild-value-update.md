---
title: "Angular에서 지연된 템플릿 업데이트 해결(Resolving Delayed Template Updates in Angular)"
date: 2024-01-08 15:00:00 +0900
comments: true
categories: angular
tags: [viewchild, template]
---

## 소개
이 글에서는 Angular 애플리케이션에서 값이 템플릿에 즉시 반영되지 않는 문제를 해결하는 방법에 대해 탐구합니다.<br/> 
보통 선언된 변수는 즉시 템플릿에 반영되지만 OnPush 변경 감지를 사용하는 경우와 같이 특정 상황에서는 그렇지 않을 수 있습니다.<br/>  
이 글은 변경 감지에 대한 깊은 설명을 생략하고 공식 Angular 문서로 대처합니다: [Angular 변경 감지 및 런타임 최적화](https://angular.io/guide/change-detection)

지연된 템플릿 업데이트 문제는 변수를 업데이트하는 상태가 Angular 주기와 일치하지 않을 때 발생합니다. 여기에는 일반적인 시나리오가 있습니다.

## Subscribe 사용
RxJS와 같은 비동기 작업을 처리할 때 값에 구독하고 이를 변수에 동기화하려고 할 때 지연된 템플릿 업데이트가 발생할 수 있습니다.

```ts
@Component({
  template: `{{ key }}`
})
export class DemoComponent {
  key!: string;

  sampleService = inject(SampleService);
  sample$ = this.sampleService.value$.pipe(
    // ...
  ).subscribe(val => {
      this.key = val; // Update not immediately reflected in the template.
  });
  // ...
}
```


## setTimeout 사용
setTimeout 내에서 값을 주입하는 경우 업데이트는 microTask에서 발생하며 이는 Angular 주기와 일치하지 않아 템플릿에 즉시 반영되지 않습니다.

```ts
@Component({
  template: `{{ key }}`
})
export class DemoComponent {
  key!: string;

  update(val: any) {
    setTimeout(() => {
      this.key = val; // Update not immediately reflected in the template.
    }, 0); // or any time value
  }
  // ...
}
```


## ViewChild에서 값 가져오기
ViewChild를 사용하여 얻은 자식 컴포넌트의 값을 템플릿에 반영하려는 경우, 이 방법은 Angular에서 권장하는 Input 및 Output 데이터 처리 방법과 일치하지 않기 때문에 즉시 업데이트되지 않을 수 있습니다.

```ts
@Component({
  template: `
    <app-child #child></app-child>
    {{ key }}
  `
})
export class ParentComponent {
  @ViewChild('child') child: AppChildComponent;

  key!: string;

  update() {
    this.key = this.child.value; // Update not immediately reflected in the template.
    this.key = this.child.getValue(); // Getting value from child function not updated yet.
  }
  // ...
}
```


## Angular 주기 밖
Angular 주기와 일치하지 않아 업데이트가 즉시 템플릿에 반영되지 않는 다양한 상황이 있습니다. 해당 상황에 대한 자세한 내용은 다른 문서를 참고하세요.

## 해결 방법
지연된 템플릿 업데이트를 해결하기 위해 `ChangeDetectorRef.markForCheck()` 또는 `detectChanges()` 를 활용하여 값의 업데이트 시기를 조절할 수 있습니다. 예를 들어, subscribe를 사용하는 경우 다음 수정으로 문제를 해결할 수 있습니다.

```ts
@Component({
  template: `{{ key }}`
})
export class DemoComponent {
  cdr = inject(ChangeDetectorRef);

  key!: string;

  sampleService = inject(SampleService);
  sample$ = this.sampleService.value$.pipe(
    // ...
  ).subscribe(val => {
      this.key = val;
      this.cdr.markForCheck(); // Immediate update reflected in the template.
  });
  // ...
}
```

ChangeDetectorRef에는 markForCheck와 detecChanges 함수를 제공하는데, markForCheck와 detectChanges의 차이는 아래의 사이트를 참고하세요.
[difference between markforcheck and detectchanges](https://stackoverflow.com/questions/41364386/whats-the-difference-between-markforcheck-and-detectchanges)
<br/>

[difference between markforcheck and detectchanges in angular](https://medium.com/@andre.schouten_ff/whats-the-difference-between-markforcheck-and-detectchanges-in-angular-fff4e5f54d34)
<br/>

[ChangeDetectorRef](https://angular.io/api/core/ChangeDetectorRef)
<br/>