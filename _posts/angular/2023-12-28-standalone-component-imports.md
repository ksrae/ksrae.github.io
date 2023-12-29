---
title: "Standalone Component에서 imports 방법 (Managing Imports in Standalone Angular Components)"
date: 2023-12-28 10:20:00 +0900
comments: true
categories: angular
tags: [angular17, update]
---

## 소개
Standalone Angular 컴포넌트에서 컴포넌트, 파이프, 디렉티브의 적절한 Imports 관리는 깨끗하고 최적화된 코드 구조를 유지하는 데 필수적입니다. <br/>이 블로그 게시물에서는 템플릿에서의 컴포넌트, 파이프, 디렉티브 다루기, CommonModule 다루기, createComponent을 사용한 동적 컴포넌트 다루기, 그리고 Standalone 컴포넌트 내에서 non-standalone 컴포넌트 다루는 최상의 방법에 대해 살펴보겠습니다.


## 1. 템플릿에서의 컴포넌트, 파이프, 디렉티브
Standalone 컴포넌트의 템플릿에서 사용 중인 모든 컴포넌트, 파이프, 디렉티브(CommonModule에 포함된 것은 제외)은 반드시 imports에 추가되어야 합니다. <br/>이는 Angular가 참조된 엔터티를 올바르게 인식하고 렌더링할 수 있도록 보장합니다.

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
템플릿에서 CommonModule에 속한 파이프 또는 디렉티브를 사용하는 경우, 최적화를 위해 각 항목을 개별적으로 imports 하는 것이 좋습니다.

#### Directive

| Name | Template | 
|:---:|:---| 
| NgClass | [ngClass] | 
| NgComponentOutlet | <ng-container *ngComponentOutlet="componentTypeExpression"></ng-container> | 
| NgFor | *ngFor |
| NgForOf | <ng-template ngFor let-item [ngForOf]="items" let-i="index" [ngForTrackBy]="trackByFn"> |
| NgIf | *ngIf |
| NgPlural | <some-element [ngPlural]="value"> |
| NgPluralCase | <ng-template ngPluralCase="=0">there is nothing</ng-template> |
| NgStyle | [ngStyle] |
| NgSwitch | <container-element [ngSwitch]="switch_expression"> |
| NgSwitchCase | <some-element *ngSwitchCase="match_expression_1">...</some-element> |
| NgSwitchDefault | <some-element *ngSwitchDefault>...</some-element> |
| NgTemplateOutlet | <ng-container *ngTemplateOutlet="greet"></ng-container> |


#### Pipe
| Name | Template | 
|:---:|:---| 
| AsyncPipe | {{ observable$ \| async }} |
| CurrencyPipe | {{ num \| currency }} |
| DatePipe | {{ str \| date }} |
| DecimalPipe | {{ num \| decimal }} |
| JsonPipe | {{ obj \| json }} |
| KeyValuePipe | {{ obj \| keyvalue }} |
| LowerCasePipe | {{ str \| lowercase }} |
| UpperCasePipe | {{ str \| uppercase }} |
| TitleCasePipe | {{ str \| titlecase }} |
| PercentPipe | {{ num \| percent }} |
| SlicePipe | {{ str \| slice: 0:4 }}



## 3. createComponent로 생성된 컴포넌트
createComponent를 사용하여 동적으로 생성된 컴포넌트는 imports에 추가할 필요가 없습니다. <br/>Angular 프레임워크가 이 방식으로 생성된 컴포넌트의 인스턴스를 자동으로 관리합니다.


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

## 4. Non-standalone 컴포넌트
Standalone 컴포넌트 내에서 non-standalone 컴포넌트를 다루기 위해서는 조심스러운 접근이 필요합니다. <br/>Standalone 컴포넌트에서 non-standalone 컴포넌트를 직접적으로 imports에 추가할 수 없기 때문에 다음 단계를 따를 수 있습니다.

- 모듈을 생성합니다.
- 모듈의 declarations에 non-standalone 컴포넌트를 추가합니다.
- 모듈의 exports에 non-standalone 컴포넌트를 추가합니다.
- Standalone 컴포넌트의 imports에 모듈을 추가합니다.


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
Standalone Angular 컴포넌트에서 Imports를 효율적으로 관리하는 것은 깨끗하고 최적화된 코드베이스를 유지하는 데 중요합니다. <br/>이 블로그 게시물에서는 템플릿에서 컴포넌트, 파이프, 디렉티브를 다루는 최상의 방법, CommonModule 다루기, createComponent를 사용하여 동적으로 컴포넌트를 생성하는 방법, 그리고 standalone 컴포넌트 내에서 non-standalone 컴포넌트를 통합하는 방법에 대해 알아보았습니다. <br/>이러한 관행을 신중하게 적용함으로써 개발자들은 견고하고 효율적인 Angular 응용 프로그램 아키텍처를 보장할 수 있습니다.

## Reference
[Common Module](https://angular.io/api/common/CommonModule)