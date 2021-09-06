---
title: "ngx-translate을 활용한 다국어 적용 (Support Multiple Languages with ngx-translate)"
comments: true
categories: angular
tags: [language, translate]
date: 2019-07-03 12:20:00 +0900
---



ngx-translate를 활용하여, 서버 요청 및 리로딩 없이 즉시 다국어를 적용합니다.<br>


## 1. 적용



우선 ngx-translate/core를 설치합니다.<br>

```
npm install @ngx-translate/core --save
```
    

## 2. 코드

### 2-1. 기본 호출 방법

##### app.module.ts

최상위 모듈에 TranslateModue.forRoot()를 import 합니다.<br>
하위 모듈에는 .forChild()를 import 합니다.<br>

```ts
import {BrowserModule} from '@angular/platform-browser';
import {NgModule} from '@angular/core';
import {TranslateModule} from '@ngx-translate/core';

@NgModule({
    imports: [
        BrowserModule,
        TranslateModule.forRoot()
    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
```


### 2-2. json으로 언어값 관리 및 AoT 기법 활용

aot기법으로 언어값을 호출할 수 있습니다. <br>
위의 app.module.ts 를 확장하여 재작성하면 아래와 같습니다.<br>
기본적으로는 유사하나 forRoot의 옵션 일부와 HttpLoader를 통한 파일 로드 과정이 추가됩니다.<br>

이 때는 http-loader를 설치해야 합니다.<br>

```
npm install @ngx-translate/http-loader --save
```

##### app.module.ts

```ts
import {BrowserModule} from '@angular/platform-browser';
import {NgModule} from '@angular/core';
import { HttpClientModule, HttpClient } from '@angular/common/http';

import { TranslateModule, TranslateLoader } from '@ngx-translate/core';
import { TranslateHttpLoader } from "@ngx-translate/http-loader";

export function HttpLoaderFactory(httpClient: HttpClient) {
  return new TranslateHttpLoader(httpClient, "assets/i18n/", ".json");
}

@NgModule({
    imports: [
        BrowserModule,
        TranslateModule.forRoot({
           loader: {
             provide: TranslateLoader,
             useFactory: HttpLoaderFactory,
             deps: [HttpClient]
           }
        }),
    ]
    bootstrap: [AppComponent]
})
export class AppModule { }
```

### lang.interface.ts

interface가 제각각이므로 언어 코드를 미리 interface화 해두는 것이 좋습니다.<br><br>
'ko', 'en'과 같이 두단어로 구성하거나 'kor', 'eng'와 같이 세단어로 구성해도 되나 <br>
만일 아래와 같이 2-2 로 구성할 경우 반드시 키값에 따옴표를 넣어야 합니다.<br><br>


```ts
export enum LanguageCode {
    'ko-kr' = '한국어',
    'en-us' = 'English',
    'zh-cn' = '简体中文'
}
```


#### language.service.ts

translateService를 그대로 사용할 수도 있으나 <br>초기 설정 및 interface와 transService를 맞추기 위한 별도의 서비스를 추가합니다.<br>

```ts

import { Injectable } from '@angular/core';

import { TranslateService } from '@ngx-translate/core';
import { LanguageCode } from '../interface/lang.interface';

@Injectable()

export class LanguageService {
    private langList: Array<string>;

    constructor(private translateService: TranslateService) {
        this.langList = Object.keys(LanguageCode);
    }

    public onInit(){
        //translateService에 변역 가능한 언어를 추가합니다.
        for(let lang of this.langList){
            this.translateService.addLangs([lang]);
        }

        // 브라우저 언어를 우선 적용합니다.
        let browserLang = this.translateService.getBrowserLang();
        let existLang = this.langList.find(lang => browserLang == lang);
        let matchingLang = existLang ? browserLang : 'ko-kr';

        // default 언어를 설정하고 사용할 언어를 정합니다.
        this.translateService.setDefaultLang(matchingLang);
        this.translateService.use(matchingLang);
    }

    // 변경할 언어를 translateService에 알립니다.
    public change(language: LanguageCode){
        let existLang = this.langList.find(lang => LanguageCode[language] == lang);
        if(existLang){
            this.translateService.use(LanguageCode[language]);
        }
    }
}
```

### 2-2. json 파일 관리

interface에서 정의한 값대로 json 파일을 생성합니다.<br>
> 예) ko.json, en.json 또는 ko-kr.json, en-us.json

생성한 파일에 "key": "해당 언어값" 과 같이 json을 구성합니다.<br>
만일 ko-kr.json 파일을 생성한다면 아래와 같이 만들어 줍니다.<br>

```json
{
    "login": "로그인",
    "email": "이메일" 
}
```

### 2-3. template에서의 활용

직접 {{ }} 에서 적용할 수 있으나 번역 내용에 tag가 들어간 경우 innerHTML에서 사용할 수도 있습니다.<br>

```html
{% raw %}
<div>{{ 'login' | translate }}</div>
{% endraw %}
```

- 언어값 내에 tag가 있는 경우 innerHTML을 사용합니다.

```html
{% raw %}
<div [innerHTML]="'login' | translate"></div>
{% endraw %}
```

> 추천드리는 방법은 innerHTML에 적용하고, text 영역에는 기본 값을 적어놓는 것입니다. 이렇게 하면 번역 키를 따라 번역 값을 일일이 확인하지 않아도 어떤 값이 적용되었는지 코드상에서 바로 파악할 수 있으며, 실제는 innerHTML의 내용이 반영되므로 코드에 영향도 없어 효율적입니다.

```html
{% raw %}
<div [innerHTML]="'login' | translate">로그인</div>
{% endraw %}
```

### 2-4. 공용 번역값 적용하기

만일 값을 유동적으로 적용하고 싶은 값이 있다면 아래와 같이 중괄호 안에 키 값을 넣습니다.<br>

```json
{
    "changeItem": "{{item}} 변경",
    "password": "비밀번호",
    "username": "유저명",
}
```

만일 template 에서 "비밀번호 변경"을 적용하고 싶다면, <br>
먼저 changeItem의 translate 파라미터에 적용하고자 하는 키 값인 item에 번역한 "password"를 정의합니다. <br>

```html
{% raw %}
<div [innerHTML]="'changeItem' | translate: {item: 'password' | translate } "></div>
{% endraw %}
```

길어지긴 했지만 위와 같은 방법으로 적용할 수 있습니다.<br>


## 3. 유의점

translate을 사용할 경우 하위 태그가 만들어지지 않는다는 점을 유의하여야 합니다.<br>

```html
{% raw %}
<div>{{ 'login' | translate }} <span>사용자 이름을 입력하세요</span></div>
{% endraw %}
```

위의 예시의 경우 span 태그가 무시되어 버립니다.<br>

> 해결 방법은 하위 태그가 필요한 경우 같은 레벨로 만들어주거나,<br>

```html
{% raw %}
<div><span>{{ 'login' | translate }}</span> <span>사용자 이름을 입력하세요</span></div>
{% endraw %}
```

> 번역에 하위 태그를 포함할 수 있습니다.
```html
{% raw %}
<div>{{ 'login' | translate }} <span>사용자 이름을 입력하세요</span></div>
{% endraw %}
```

```json
{
    "login": "로그인 <span>사용자 이름을 입력하세요</span>"
}
```


이 외에도 TranslateService에는 다양한 함수가 존재하니 다음 시간에 보다 자세히 다루어 보도록 하겠습니다.<br>

## 참고 사이트
- [ngx-translate 사이트](https://github.com/ngx-translate/core)