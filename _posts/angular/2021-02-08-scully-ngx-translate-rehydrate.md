---
title: "Scully에 ngx-translate 적용 시 rehydration 방지 (Prevent rehydration using ngx-translate on scully)"
date: 2021-02-08 14:37:00 +0900
comments: true
categories: angular
tags: [ngx-translate, rehydrate, scully]
---


## 과제
1. scully에 ngx-translate 적용하기<br/>
2. 실행 후 언어를 변경하고 refresh 했을 때 rehydration 현상 확인<br/>
3. 처음부터 변경된 언어로 작성된 static html을 화면에 노출하기<br/>




## 계획
1. LocalStorage를 활용하여 변경된 언어를 저장한다.<br/>
2. 리로드 시 페이지 로딩 첫 시점부터 변경된 언어가 적용된 html 이 보여야 한다.<br/>



## DEMO 작성
#### 다국어 적용 (ngx-translate 적용)
기존 코드에 ngx-translate을 설치합니다. 다국어 파일을 httploader를 통해 불러오므로 함께 설치합니다.<br/>

```
npm i @ngx-translate/core @ngx-translate/http-loader
```

app.module에서 ngx-translate 사용을 설정합니다. 이 때, defaultLanguage를 localstorage에서 가져온 값으로 설정합니다. 

```tsx
...
export function createTranslateLoader(http: HttpClient) {
  return new TranslateHttpLoader(http, './assets/i18n/', '.json');
}
 
 
...
  imports: [
    BrowserModule.withServerTransition({ appId: 'serverApp' }),
    HttpClientModule,
    TranslateModule.forRoot({
      defaultLanguage: localStorage.getItem('lang') ?? 'ko',
      loader: {
          provide: TranslateLoader,
          useFactory: (createTranslateLoader),
          deps: [HttpClient]
      }
  }),
```

assets/i18n에 다국어를 위해 ko.json과 en.json을 만들고 값을 넣습니다. (한국어를 길게 넣은 이유는 테스트 결과 확인 시 잘 보이도록 하기 위함입니다.)

```json
// ko.json
{
  "lang": "한국어한국어한국어한국어한국어한국어"
}
// en.json
{
  "lang": "eng"
}
```

template에 언어 변경 버튼 및 텍스트를 배치합니다.

```html
<button (click)="change('ko')">KO</button>
<button (click)="change('en')">EN</button>
<p>{{'a' | translate}}</p>
```

component에는 change 함수를 추가합니다. 이 때, constructor에 translateService를 선언해야 합니다.

```tsx
change(type: string) {
  localStorage.setItem('lang', type);
  this.translateService.use(type);
}
```

이 상태로 scully를 실행하여 영문으로 변경한 뒤 새로고침을 해보면, static으로 생성된 국문 html 파일이 먼저 생성된 후 변경된 언어가 적용됨을 확인할 수 있습니다. 즉, 목적에서 짚은 현상이 발견 되는 것을 확인할 수 있습니다.



#### ngx-translate-router 적용
위의 목적을 달성해줄 라이브러리를 설치합니다.

```
npm i @gilsdav/ngx-translate-router
```

app-routing.module에 LocalizeRouterModule을 선언합니다. 이 때, 반드시 TranslateService와 RouterModule 이 먼저 선언 되어야 합니다.

```tsx
  TranslateModule.forRoot({
    defaultLanguage: localStorage.getItem('lang') ?? 'ko',
    loader: {
        provide: TranslateLoader,
        useFactory: (createTranslateLoader),
        deps: [HttpClient]
    }
}),
RouterModule.forRoot(routes),
LocalizeRouterModule.forRoot(routes),
AppRoutingModule,
```

이제 scully를 실행하여 영문으로 변경한 뒤 새로고침을 해보면 static 파일이 영문 html이 생성되어 즉시 로딩 된 것을 확인할 수 있습니다. (국문도 동일합니다.)


## 결론
@ gilsdav / ngx-translate-router는 localized 방식으로 동작합니다. 즉, 모든 언어의 라우터를 생성한 뒤 해당 라우터만 사용하는 방식입니다.<br/>
접근하는 시점에서 라우터를 가로 채고 localized된 route를 redirectTo 경로로 변환합니다.  예를 들어 /home은 /ko/home, /en/home 중 ngx-translate이 선택한 경로로 변환합니다. <br/>
라우터 변환을 일반 애플리케이션 변환과 분리하기 위해 접두사를 사용합니다. escapePrefix를 사용하여 접두사가 제거되고 세그먼트가 번역되지 않도록 할 수 있습니다. <br/>

```tsx
  let routes = [
  // localized를 적용합니다.
  { path: 'home', component: HomeComponent },
  // 적용하지 않습니다.
  { path: 'login', component: LoginComponent, data: { skipRouteLocalization: true } }
    // route에는 적용하지 않으나 redirect에만 적용합니다.
  { path: 'logout', redirectTo: 'login', data: { skipRouteLocalization: { localizeRedirectTo: true } } }
];
```

이 라이브러리는 ngx-translate이 선택한 locales를 자동으로 인식하여 동작합니다.

```tsx
this.translate.setDefaultLang();
this.translate.use();
```




## 참고 사이트
[ngx-translate-router](https://github.com/gilsdav/ngx-translate-router)

