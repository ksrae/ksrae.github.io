---
title: "Angular Standalone Component와 새로운 의존성 주입 방식"
date: 2024-09-07 19:27:00 +0900
comments: true
categories: angular
tags: [standalone, inject, ondestroy, afterrender, afternextrender]
---

# Angular Standalone Component와 새로운 의존성 주입 방식
Angular는 최신 버전에서 개발자 경험을 크게 개선하는 여러 기능을 도입했습니다. 그 중에서도 Standalone Component와 inject API는 Angular 애플리케이션의 구조와 의존성 관리 방식을 변화시키는 핵심 요소입니다. 이 글에서는 이러한 변화들이 Angular 개발에 미치는 영향을 살펴보고, 최신 의존성 주입 방식인 inject와 컴포넌트 생명주기 관리에 관한 새로운 패턴을 소개하겠습니다.

## Standalone Component: 모듈 종속성 없이 컴포넌트 사용
기존 Angular 프로젝트에서 모든 컴포넌트는 하나 이상의 NgModule에 종속되었습니다. 그러나 Angular 15부터는 Standalone Component 개념이 도입되어 모듈 종속성을 줄일 수 있게 되었습니다. 이를 통해 개발자는 더 간결하고 독립적인 컴포넌트를 만들 수 있습니다.

### Standalone Component의 주요 옵션
Standalone Component를 설정할 때 다양한 옵션을 사용할 수 있습니다:

- standalone: true: 컴포넌트를 모듈 없이 독립적으로 정의하는 옵션입니다.
- selector: 컴포넌트를 HTML에서 사용할 태그 이름을 정의합니다.
- templateUrl 및 template: HTML 템플릿을 외부 파일이나 문자열로 설정합니다.
- styleUrl: 기존에는 스타일 파일을 반드시 styleUrls에 배열로 정의했으나 이제 단일 설정이 가능합니다.
- imports: 다른 Angular 모듈, 디렉티브, 파이프 등을 불러옵니다.
- providers: 컴포넌트에서 사용할 서비스나 의존성 주입을 정의합니다.
- schema: 상호작용할지를 결정하는 설정입니다. 주로 컴파일러가 컴포넌트를 분석하고, 템플릿에서 허용할 수 있는 구조적 요소나 규칙을 정의하는 데 사용됩니다. NO_ERRORS_SCHEMA, CUSTOM_ELEMENTS_SCHEMA 등이 있습니다.

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  standalone: true,
  selector: 'app-example',
  templateUrl: './example.component.html',
  styleUrl: './example.component.css',
  imports: [CommonModule],
  providers: [HttpService]
})
export class ExampleComponent {}
```


## 의존성 주입의 진화: inject API
기존 Angular에서 의존성 주입은 **생성자 주입(Constructor Injection)**을 통해 이루어졌습니다. 컴포넌트 또는 서비스의 생성자에 의존성을 주입함으로써 Angular의 DI(Dependency Injection) 시스템이 자동으로 객체를 관리했습니다. 예를 들어, 다음과 같은 코드가 일반적이었습니다:

```typescript
import { Component } from '@angular/core';
import { SomeService } from './some.service';

@Component({
  selector: 'app-example',
  templateUrl: './example.component.html'
})
export class ExampleComponent {
  constructor(private someService: SomeService) {}
}
```
이 패턴은 여전히 유용하지만, Angular 15부터는 더 유연한 방식으로 의존성을 주입할 수 있는 inject API가 도입되었습니다.

### inject API의 특징과 사용 예시
inject를 사용하면 의존성을 클래스 생성자가 아닌, 메서드나 함수 내부에서 주입할 수 있습니다. 이를 통해 더 유연한 코드 작성이 가능해졌으며, 특히 비동기 작업이나 동적 서비스 관리에 강력한 도구가 됩니다.

```typescript
import { Component, inject } from '@angular/core';
import { SomeService } from './some.service';

@Component({
  selector: 'app-example',
  templateUrl: './example.component.html'
})
export class ExampleComponent {
  private someService = inject(SomeService);

  someMethod() {
    this.someService.performAction();
  }
}
```

### 왜 inject를 사용할까?
- 유연성과 간결성: 메서드 내부에서 의존성을 주입받을 수 있어 코드가 간결해집니다.
- 테스트 용이성: 특정 메서드에서만 의존성을 주입받는 구조 덕분에 테스트와 재사용이 쉬워집니다.
- 비동기 작업 최적화: 복잡한 비동기 작업에서 의존성 주입을 더욱 유연하게 관리할 수 있습니다.
- Zoneless 변경 감지: Angular의 새로운 Zoneless 변경 감지 방식과의 통합을 지원하여 성능을 더욱 최적화할 수 있습니다.

## 컴포넌트 생명주기 관리의 진화: DestroyRef와 더 나은 리소스 해제
Angular에서 컴포넌트가 파괴될 때 주로 ngOnDestroy 메서드를 사용하여 구독을 해제하거나 자원을 정리합니다. 하지만 Angular 15에서는 더 간편한 방식인 **DestroyRef**를 도입하여 생명주기 관리가 더욱 직관적으로 변화했습니다.

### DestroyRef 사용 예시
DestroyRef는 의존성 주입을 통해 제공되며, RxJS의 takeUntil 연산자와 함께 사용하면 컴포넌트 파괴 시 구독을 자동으로 해제할 수 있습니다.

```typescript
import { Component, inject, OnInit } from '@angular/core';
import { DestroyRef } from '@angular/core';
import { interval } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

@Component({
  selector: 'app-example',
  template: `<h1>DestroyRef Example</h1>`
})
export class ExampleComponent implements OnInit {
  private destroyRef = inject(DestroyRef);
  private numbers$ = interval(1000).pipe(takeUntil(this.destroyRef.onDestroy()));

  ngOnInit() {
    this.numbers$.subscribe(num => console.log('Number:', num));
  }
}
```
이처럼 **DestroyRef**는 컴포넌트가 파괴될 때 리소스를 자동으로 정리해주어, 메모리 누수를 방지하고 코드 복잡성을 줄입니다.

## 렌더링 후 작업: afterRender와 afterNextRender
Angular는 렌더링 후에 발생하는 작업을 처리하기 위해 **ngAfterViewInit**과 ngAfterViewChecked 같은 생명주기 훅을 제공합니다. 그러나 최근에는 보다 세밀한 제어를 가능하게 하는 **afterRender**와 afterNextRender 메서드가 도입되었습니다.

### afterRender: 렌더링 후 작업 처리
afterRender는 뷰가 렌더링된 직후 호출되며, DOM 요소가 완전히 준비된 시점에서 스타일 적용, DOM 조작 등을 수행하는 데 유용합니다.

```typescript
@Component({
  standalone: true,
  selector: 'my-chart-cmp',
  template: `<div #chart>{{ ... }}</div>`,
})
export class MyChartCmp {
  @ViewChild('chart') chartRef: ElementRef;
  chart: MyChart|null;

  constructor() {
    afterNextRender(() => {
      this.chart = new MyChart(this.chartRef.nativeElement);
    }, {phase: AfterRenderPhase.Write});
  }
}
```

### afterNextRender: 다음 렌더링 주기 후 작업 처리
afterNextRender는 다음 렌더링 주기가 완료된 후에 호출되며, 데이터 변경 후 UI 업데이트 작업을 효율적으로 처리할 수 있습니다.

```typescript
@Component({
  standalone: true,
  selector: 'my-cmp',
  template: `<span #content>{{ ... }}</span>`,
})
export class MyComponent {
  resizeObserver: ResizeObserver|null = null;
  @ViewChild('content') contentRef: ElementRef;

  constructor() {
    afterNextRender(() => {
      this.resizeObserver = new ResizeObserver(() => {
        console.log('Content was resized');
      });

      this.resizeObserver.observe(this.contentRef.nativeElement);
    }, {phase: AfterRenderPhase.Write});
  }

  ngOnDestroy() {
    this.resizeObserver?.disconnect();
    this.resizeObserver = null;
  }
}
```


