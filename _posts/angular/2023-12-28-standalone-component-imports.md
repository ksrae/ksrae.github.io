---
title: "Standalone Component에서 imports 방법 (Managing Imports in Standalone Angular Components)"
date: 2023-12-28 10:20:00 +0900
comments: true
categories: angular
tags: [standalone, update]
---

## 소개
Standalone Component에서 Component, Pipe, Directive 의 적절한 Imports 관리는 깨끗하고 최적화된 코드 구조를 유지하는 데 필수적입니다. <br/>이 글에서는 Template에서의 Component, Pipe, Directive 다루기, CommonModule 최적화 하기, createComponent을 사용한 동적 Component 다루기, 그리고 Standalone Component 에서 non-standalone Component 다루는  방법에 대해 살펴보겠습니다.


## 1. Template에서의 Component, Pipe, Directive
Standalone Component의 Template에서 사용 중인 모든 Component, Pipe, Directive는 반드시 imports에 추가되어야 합니다. <br/>이는 Angular가 참조된 엔터티를 올바르게 인식하고 렌더링할 수 있도록 보장합니다.<br/>Pipe와 Directive 가 CommonModule에 속한 경우에 대하여는 2번 항목을 참조하세요. <br/> 아래의 예제는 일반적인 상황(Pipe와 Directive가 CommonModule 에 속하지 않은 경우) 에서 imports 하는 방법을 보여줍니다.

```ts
// app.component.ts

@Component({
  template: `
    <div [ngClass]="value | customPipe" customDirective>
      <app-child></app-child>
    </div>
  `,
  standalone: true,
  imports: [
    CustomPipe,
    CustomDirective,
    AppChildComponent
  ]
})
export class AppComponent {}
```


## 2. CommonModule
Template에서 CommonModule에 속한 Pipe 또는 Directive를 사용하는 경우, 최적화를 위해 각 항목을 개별적으로 imports 하는 것이 좋습니다. <br/>예를 들어 자주 사용하는 ngIf, ngFor나 async pipe의 경우 CommonModule을 imports 하거나 또는 CommonModule에 속한 해당 항목을 imports 합니다. <br/>가급적이면 성능 최적화를 위해 CommonModule을 직접 imports 하지말고, 정확하게 필요한 항목만 imports 하는 것을 추천합니다. <br/> 아래는 CommonModule에 포함된 Directive와 Pipe의 목록 입니다.

#### Directive

- NgClass
- NgComponentOutlet
- NgFor
- NgForOf
- NgIf
- NgPlural
- NgPluralCase
- NgStyle
- NgSwitch
- NgSwitchCase
- NgSwitchDefault
- NgTemplateOutlet


#### Pipe
- AsyncPipe
- CurrencyPipe
- DatePipe
- DecimalPipe
- JsonPipe
- KeyValuePipe
- LowerCasePipe
- UpperCasePipe
- TitleCasePipe
- PercentPipe
- SlicePipe



## 3. createComponent로 생성된 Component
createComponent를 사용하여 동적으로 생성된 Component는 imports에 추가할 필요가 없습니다. <br/>Angular 프레임워크가 이 방식으로 생성된 Component의 인스턴스를 자동으로 관리합니다.


```ts
// app.component.ts

@Component({
  template: ``,
  standalone: true,
  imports: []
})
export class AppComponent {
  viewContainer = inject(ViewContainerRef);
  componentRef = this.viewContainer.createComponent(AddComponent);
}
```

## 4. Non-standalone Component
Standalone Component 내에서 non-standalone Component를 다루기 위해서는 조심스러운 접근이 필요합니다. <br/>Standalone Component에서 non-standalone Component를 직접적으로 imports에 추가할 수 없기 때문에 다음 단계를 따를 수 있습니다.

- Module을 생성합니다.
- Module의 declarations에 non-standalone Component를 추가합니다.
- Module의 exports에 non-standalone Component를 추가합니다.
- Standalone Component의 imports에 Module을 추가합니다.


### 예시
더 자세한 내용을 알아보기 위해 다음의 예시를 살펴봅시다.

#### Non Standalone Component

```ts
// shared.component.ts

@Component({
  selector: 'shared-component',
  template: `<p>SHARED COMPONENT</p>`,
})
export class SharedComponent {}
```

#### Module

```ts
// shared.module.ts

@NgModule({
  declarations: [
    SharedComponent
  ],
  exports: [
    SharedComponent
  ]
})
export class SharedModule {}
```

#### Standalone Component

```ts
// app.component.ts

@Component({
  selector: 'app-root',
  template: `<shared-component></shared-component>`,
  standalone: true,
  imports: [
    SharedModule
  ]
})
export class AppComponent
```


## 결론
Standalone Component에서 Imports를 효율적으로 관리하는 것은 깨끗하고 최적화된 코드베이스를 유지하는 데 중요합니다. <br/>이러한 방법을 신중하게 적용하면 견고한 코드 및 효율적인 성능을 보장할 수 있습니다.

## Reference
[Common Module](https://angular.io/api/common/CommonModule)