---
title: "Standalone의 개념과 main.ts의 중요성"
date: 2024-09-05 22:04:00 +0900
comments: true
categories: angular
tags: [standalone, maints]
---

# Angular Standalone: Standalone 개념과 main.ts의 중요성
Angular는 애플리케이션 아키텍처를 간소화하고 성능을 최적화하기 위한 여러 기능을 도입했습니다. 그중에서도 가장 중요한 변화인 Standalone는 Angular 애플리케이션 개발 방식에 큰 변화를 가져온 중요한 기능입니다. 기존의 NgModule 기반 아키텍처에서 벗어나 컴포넌트를 독립적으로 선언하고 사용할 수 있게 함으로써, 모듈 종속성을 최소화하고 애플리케이션 초기화 과정도 간결하게 만듭니다. 본 글에서는 Angular Standalone의 개념과 이를 왜 사용해야 하는지, 그리고 main.ts에서의 초기 로딩 작업의 필요성에 대해 다뤄보겠습니다.

# Standalone 컴포넌트란?
전통적으로 Angular 애플리케이션은 NgModule을 기반으로 작동했습니다. 각 기능은 하나 이상의 모듈에 정의되고, 컴포넌트, 서비스, 디렉티브 등은 모두 모듈의 일부로 선언되었습니다. 그러나 이 방식은 작은 애플리케이션에서는 과도한 복잡성을 초래할 수 있으며, 특히 컴포넌트나 서비스를 재사용할 때 모듈을 함께 관리해야 한다는 점에서 불편함이 있었습니다.

이러한 문제를 해결하기 위해 Angular는 Standalone 컴포넌트 개념을 도입했습니다. Standalone 컴포넌트는 기존의 모듈 없이도 독립적으로 정의될 수 있는 컴포넌트로, 이를 통해 불필요한 모듈 선언을 줄이고 애플리케이션의 구조를 간소화할 수 있습니다. Standalone 컴포넌트를 사용하면, 컴포넌트가 필요로 하는 모듈, 디렉티브, 파이프 등을 직접 imports에 선언할 수 있습니다. 결과적으로 더 가볍고 빠른 애플리케이션을 만들 수 있게 됩니다.

```typescript
@Component({
  standalone: true,
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
  imports: [CommonModule, FormsModule] // 필요한 모듈을 직접 가져옴
})
export class AppComponent { }
```

# main.ts의 역할
Angular에서 애플리케이션의 진입점(entry point)은 main.ts 파일입니다. 이 파일은 브라우저에서 애플리케이션을 부트스트랩하는 역할을 하며, 애플리케이션이 처음 실행될 때 필요한 초기 설정을 담당합니다. 전통적인 Angular 방식에서는 AppModule을 부트스트랩하는데 사용되었지만, Standalone 컴포넌트가 도입되면서 모듈 없이 AppComponent와 같은 루트 컴포넌트를 직접 부트스트랩할 수 있게 되었습니다.

이 과정에서 main.ts는 필수적인 초기 설정을 담당하며, 특히 라우팅, HTTP 클라이언트 설정, 다국어 지원 등의 기능을 이곳에서 정의할 수 있습니다. 즉, 모듈의 필요성이 줄어든 대신, 각 기능을 직접 main.ts에서 로드하고 제공자(provider)로 선언하여 애플리케이션의 전역 설정을 수행해야 합니다.

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch(err => console.error(err));
```

위와 같은 코드에서 bootstrapApplication은 AppComponent를 애플리케이션의 루트 컴포넌트로 설정하고 브라우저에 애플리케이션을 로드합니다.

# main.ts에서의 초기 로딩 작업
main.ts에서 수행하는 초기 로딩 작업은 애플리케이션의 동작을 제어하는 중요한 역할을 합니다. 특히 HTTP 클라이언트 설정, 라우팅 및 다국어 처리 등의 작업은 이 파일에서 이루어져야 합니다. Standalone 컴포넌트 방식에서는 이러한 설정들을 모듈 대신 providers를 통해 전달하게 되며, Angular는 이를 전역 서비스로 사용하게 됩니다.

## providers


## HTTP 클라이언트 설정
Angular의 HttpClient는 서버와의 통신을 담당하는 핵심 모듈입니다. Standalone 애플리케이션에서도 서버와의 통신을 위해 HttpClientModule을 사용할 수 있으며, 이는 main.ts에서 초기화할 수 있습니다. 예를 들어, XSRF 보호 설정, JSONP 지원 등을 main.ts에서 선언할 수 있습니다.

```typescript
bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(
      withJsonpSupport(),
      withXsrfConfiguration({
        cookieName: 'XSRF-TOKEN',
        headerName: 'X-XSRF-TOKEN'
      }),
      withNoXsrfProtection()
    )
  ]
});
```

### withJsonpSupport()
JSONP는 주로 CORS 이슈를 해결하기 위해 사용되는 방식으로, 다른 도메인의 자원을 요청할 때 서버에서 CORS 설정이 되어 있지 않은 경우 JSONP를 사용해 응답을 받을 수 있습니다. withJsonpSupport()는 Angular의 HttpClient에 JSONP 지원을 추가합니다. 이는 필요한 경우에만 활성화하면 됩니다.

### withXsrfConfiguration()
이 옵션은 XSRF (Cross-Site Request Forgery) 보호를 위한 설정입니다. 기본적으로 Angular는 쿠키와 헤더를 이용해 XSRF 보호를 지원합니다. 이 함수는 보호에 사용되는 쿠키 이름과 헤더 이름을 커스터마이즈할 수 있게 해줍니다.

- cookieName: 'TOKEN': CSRF 토큰이 담긴 쿠키의 이름을 설정합니다. 기본값은 XSRF-TOKEN입니다.
- headerName: 'X-TOKEN': 요청 시 서버로 보내는 XSRF 토큰을 포함할 헤더 이름을 설정합니다. 기본값은 X-XSRF-TOKEN입니다.

### withNoXsrfProtection()
XSRF 보호를 비활성화하는 옵션입니다. 보안이 중요한 API 호출에서 사용하면 안 되며, XSRF 토큰을 보내지 않아도 되는 상황에서 사용할 수 있습니다. 예를 들어, 외부 API와 통신할 때 사용할 수 있습니다.


## Zoneless: 새로운 Change Detection 방식

### Zoneless의 개념과 변화
기존에 zone.js 라이브러리를 사용해 Change Detection을 자동으로 트리거했습니다. 하지만 zone.js는 성능과 개발 경험 측면에서 단점이 존재했습니다. 이러한 문제를 해결하기 위해 Angular는 18 버전부터 Zoneless라는 새로운 방식을 도입했습니다. Zoneless는 zone.js에 의존하지 않는 방식으로, 성능 최적화와 더불어 여러 이점을 제공합니다.<br/>
<br/>
Zoneless를 사용하면 더 빠른 초기 렌더링과 런타임, 더 작은 번들 크기, 간소화된 디버깅 경험을 얻을 수 있습니다. Zoneless 기능을 사용하려면 애플리케이션 부트스트랩 시 provideExperimentalZonelessChangeDetection을 추가하고, angular.json 파일에서 zone.js를 제거하면 됩니다.

```typescript

bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});
```
Zoneless 방식에서는 기존 ChangeDetectionStrategy.OnPush를 사용하는 컴포넌트들이 대부분 호환되며, signal을 활용한 상태 관리 방식으로 UI 업데이트가 이루어집니다.

```typescript
@Component({
  standalone: true,
  template: `
    <h1>Hello from {{ name() }}!</h1>
    <button (click)="handleClick()">Go Zoneless</button>
  `,
})
export class AppComponent {
  protected name = signal('Angular');

  handleClick() {
    this.name.set('Zoneless Angular');
  }
}
```

이 방식에서는 상태 변경이 발생할 때 Angular가 자동으로 Change Detection을 트리거하지 않고, 신호(signal)가 업데이트될 때만 UI가 변경됩니다. 또한 새로운 스케줄러가 연속적인 Change Detection을 하나로 합쳐 최적화된 성능을 제공합니다.

### Zoneless의 장점
마이크로 프론트엔드 및 다른 프레임워크와의 상호 운용성 향상: Zoneless는 다른 프레임워크와 함께 사용할 때 더욱 유연합니다.
- 더 빠른 초기 렌더 및 런타임: 불필요한 zone.js 트리거를 제거함으로써 성능이 개선됩니다.
- 번들 크기 감소: zone.js 제거로 더 작은 번들 크기를 유지할 수 있습니다.
- 간결한 디버깅 경험: 스택 트레이스가 더 간결하고 명확해집니다.
- 단순한 디버깅 및 유지보수: 디버깅이 직관적으로 변하며, 스케줄링으로 인해 Change Detection이 여러 번 트리거되는 문제를 해결할 수 있습니다.

### Zoneless 전환 방법
Zoneless로의 전환은 비교적 간단하며, 특히 기존에 ChangeDetectionStrategy.OnPush를 사용한 컴포넌트는 큰 수정 없이 전환할 수 있습니다. bootstrapApplication에서 provideZoneChangeDetection을 사용해 이벤트 병합 기능을 활성화할 수도 있습니다.

```typescript
bootstrapApplication(AppComponent, {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true })
  ]
});
```
또한 Zoneless 애플리케이션에서는 native async/await를 사용할 수 있어, zone.js에서 필요했던 다운레벨링이 필요 없게 됩니다. 이로 인해 번들 크기와 디버깅 성능이 개선됩니다.



## provideRouter 라우팅 설정
Angular 라우팅 설정도 main.ts에서 수행됩니다. Standalone 방식에서는 RouterModule 대신 provideRouter를 사용하여 라우팅 설정을 제공합니다. 이는 라우트 구성과 함께 다양한 추가 설정을 통해 네비게이션 동작을 제어할 수 있습니다.

``` typescript
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(
      appRoutes,
      withPreloading(PreloadAllModules),  
      withDebugTracing(),  
      withDisabledInitialNavigation(),
      withEnabledBlockingInitialNavigation(),
      withInMemoryScrolling(),
      withRouterConfig(),
    )
  ]
});
```

### withPreloading()
이 옵션은 미리 로딩할 모듈을 지정하는 데 사용됩니다.

- NoPreloading: 모듈을 미리 로드하지 않으며, 사용자가 해당 라우트로 이동할 때만 로딩됩니다.
- PreloadAllModules: 애플리케이션이 로드된 후 나중에 필요할 수 있는 모든 모듈을 미리 로드합니다. 대형 애플리케이션에서 페이지 로딩 속도를 향상시키는 데 유용합니다.

### withDebugTracing()
이 옵션을 사용하면 Angular 라우터가 실행되는 동안 발생하는 다양한 이벤트를 콘솔에 출력해 디버깅할 수 있습니다. 라우팅 흐름을 추적하고 싶을 때 유용합니다.

### withDisabledInitialNavigation()
라우터가 애플리케이션 부트스트랩 후 처음 로딩될 때, 자동으로 네비게이션이 시작되지 않도록 비활성화합니다. 복잡한 초기화 논리나 타이밍 이슈가 있는 애플리케이션에서 사용할 수 있습니다.

### withEnabledBlockingInitialNavigation()
부트스트랩이 완료되기 전에 네비게이션이 완료될 때까지 기다리도록 설정하는 옵션입니다. 서버사이드 렌더링 (SSR)에서는 이 옵션을 사용하는 것이 필수적입니다. 이 옵션을 사용하면 초기 네비게이션이 완료될 때까지 앱 로드가 지연됩니다.

### withInMemoryScrolling()
이 옵션은 스크롤 동작을 제어합니다. 예를 들어, 페이지 이동 시 이전 스크롤 위치를 유지하거나 페이지 상단으로 이동하는 등의 동작을 정의할 수 있습니다.

- anchorScrolling: 'enabled': 앵커를 통한 스크롤을 활성화합니다.
- scrollPositionRestoration: 'enabled' | 'top' | 'disabled': 페이지 이동 후 스크롤 위치를 복원하는 방식을 설정합니다.
    - 'enabled': 이전 페이지의 스크롤 위치를 복원합니다.
    - 'top': 새로 로드된 페이지의 상단으로 스크롤합니다.
    - 'disabled': 스크롤 위치를 복원하지 않습니다.

### withRouterConfig()
라우터의 전체 동작을 더 세밀하게 조정할 수 있는 옵션입니다.

- canceledNavigationResolution: 'replace' | 'computed': 네비게이션이 취소된 경우 URL을 어떻게 처리할지 결정합니다.
- onSameUrlNavigation: 'reload' | 'ignore': 동일한 URL로 네비게이션할 때 어떻게 처리할지를 결정합니다. 'reload'는 해당 URL을 다시 로드하고, 'ignore'는 네비게이션을 무시합니다.
- paramsInheritanceStrategy: 'always' | 'emptyOnly': 자식 라우트에서 부모의 파라미터를 상속하는 방식을 설정합니다. 'always'는 언제나 상속하고, 'emptyOnly'는 자식 라우트에 파라미터가 없을 때만 상속합니다.
- urlUpdateStrategy: 'deferred' | 'eager': URL이 업데이트되는 시점을 결정합니다. 'deferred'는 네비게이션이 완료된 후에 URL을 업데이트하고, 'eager'는 네비게이션 시작 시 바로 URL을 업데이트합니다.

## importProvidersFrom 모듈 호출
importProvidersFrom은 모듈 기반 시스템에서 사용하던 기존 Angular 모듈을 standalone 애플리케이션에 쉽게 통합할 수 있도록 도와줍니다. 이를 통해 기존 모듈을 가져올 수 있습니다. importProvidersFrom에 전달할 수 있는 대표적인 항목은 다음과 같습니다.

- HttpClientModule: HTTP 통신을 지원하는 모듈입니다. HTTP 요청, 응답을 처리하는 데 필요한 기능을 제공합니다.
BrowserModule: 브라우저에서 애플리케이션을 실행하기 위해 필요한 필수 모듈입니다. 보통 루트 모듈에서 한 번만 가져옵니다.
- TranslateModule.forRoot(): 다국어 지원을 위한 모듈로, 언어 번역을 위한 설정을 여기에 정의합니다. 예제에서는 TranslateLoader와 함께 번역 파일을 로드하는 로직이 담겨 있습니다.
이 외에도 필요한 Angular 모듈을 추가할 수 있습니다. 예를 들어, FormsModule, ReactiveFormsModule을 포함해 폼을 다루거나 RouterModule로 라우팅을 설정할 수 있습니다.

```typescript
bootstrapApplication(AppComponent, {
  providers: [
    importProvidersFrom([
      HttpClientModule,
      TranslateModule.forRoot({
        loader: {
          provide: TranslateLoader,
          useFactory: (http: HttpClient) => new TranslateHttpLoader(http, './assets/i18n/', '.json'),
          deps: [HttpClient]
        }
      })
    ])
  ]
});
```


# 결론: Standalone 방식의 장점과 main.ts의 중요성
Standalone 컴포넌트는 기존의 NgModule 기반 아키텍처에서 벗어나 더 유연하고 간결한 개발 방식을 제공합니다. 이를 통해 애플리케이션 구조를 최적화하고 불필요한 복잡성을 제거할 수 있습니다. 하지만 Standalone 방식을 사용할 때는 main.ts에서 초기화 작업을 신중히 설정해야 하며, 특히 전역 설정(예: HTTP 클라이언트, 라우팅, 다국어 지원 등)은 모두 이 파일에서 정의됩니다.

이 방식은 모듈에 의존하지 않고 컴포넌트를 독립적으로 개발하고 사용할 수 있게 하며, 특히 대규모 애플리케이션에서 모듈 관리의 복잡성을 줄이는 데 큰 도움이 됩니다. Angular의 Standalone 컴포넌트와 이를 지원하는 main.ts 설정 방식은 앞으로 Angular 애플리케이션의 개발 방식을 보다 효율적이고 직관적으로 변화시킬 것입니다.