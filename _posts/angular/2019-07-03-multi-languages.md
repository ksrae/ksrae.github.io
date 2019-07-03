---
title: "다국어 적용 (Multi Languages)"
comments: true
categories: angular
tags: [language]
date: 2019-07-03 12:20:00 +0900
---




    목적: 사이트의 다국어를 페이지 전환없이 즉시 적용되도록 하는 구조 설정.
    적용방법: 반드시 angular2의 ngx-translate를 install 하여야 한다.
    참고사이트: https://github.com/ngx-translate/core



## 1. 기본
#### app.module.ts
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

## 2. json으로 언어값 관리 및 AoT 기법 활용
### 2-1. AoT 기법 활용

위의 app.module.ts 를 확장한다.

#### app.module.ts
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

language.service

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
        for(let lang of this.langList){
            this.translateService.addLangs([lang]);
        }

        let browserLang = this.translateService.getBrowserLang();
        let existLang = this.langList.find(lang => browserLang == lang);
        let matchingLang = existLang ? browserLang : 'ko';

        this.translateService.setDefaultLang(matchingLang);
        this.translateService.use(matchingLang);
    }

    public changeLanguage(language: LanguageCode){
        let existLang = this.langList.find(lang => LanguageCode[language] == lang);
        if(existLang){
            this.translateService.use(LanguageCode[language]);
        }
    }
}
```

### 2-2. json 파일 관리
- 언어키별로 json파일을 생성. 예) ko.json, en.json 또는 ko-kr.json, en-us.json
-  json형태의 key, value로 값을 추가
```json
{
    "login": "로그인",
    "email": "이메일" 
}
```

## 3. template에서의 활용

```
<div>{{ 'login' | translate }}</div>
```

- 언어값 내에 tag가 있는 경우 innerHTML을 사용한다.

```xml
<div [innerHTML]="'login' | translate"></div>
```
