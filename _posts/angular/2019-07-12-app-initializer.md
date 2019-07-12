---
title: "최초 component 보다 빠르게 함수 실행하기 (App Initializer 활용)"
date: 2019-07-12 15:01:00 +0900
comments: true
categories: angular
tags: [initializer, factory, module]
---


앱이 실행되는 시점에서 component보다 빠르게 실행하는 함수를 만들어 보고자 합니다.

프로젝트를 만들다보면 종종 component보다 빠르게 함수를 실행하여 값을 설정하거나 불러올 필요가 있습니다.
예를 들면, json 값을 가져와야 하거나 로그를 남길지의 여부를 세팅한다거나 하는 상황이 있습니다.

이럴 때는 APP_INITIALIZER 를 활용하면 됩니다.

우선 동작할 함수를 만듭니다. 여기에서는 service를 하나 만들어 json 파일을 불러오겠습니다.

```ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http' 

@Injectable({
    providedIn: 'root'
})
export class ConfigService {

    constructor(private httpClient: HttpClient) {

    }

    public async get() {
        return new Promise((resolve, reject) => {
            this.httpClient.get('/assets/config.json',   
            ).toPromise().then((result) => {
                resolve(result);
            }).catch(err => {
                resolve(null);
            });
        });
    }
}

```

get 함수를 호출하면 config.json 파일을 읽어 Promise로 resolve하는 service를 만들었습니다.





이제 처음 실행되는 module (일반적으로 app.module)을 수정합니다.


```ts

import { NgModule, APP_INITIALIZER } from '@angular/core';
import { ConfigService } from '../services/config.service';

export function getConfig(configService: ConfigService) {
  return () => configService.get();
}

@NgModule({
      providers: [{
            provide: APP_INITIALIZER,
            useFactory: getConfig,
            deps: [ConfigService],
            multi: true
      }]
})

```

먼저 @angular/core에서 APP_INITIALIZER를 import 하였습니다.
그리고 사용할 service를 import 합니다. 여기에서는 config.service를 호출합니다.

service는 더 이상 module에서 호출하지 않도록 변경되었으나 이 경우 module에서 service를 사용하여야 하므로 service도 import 하였습니다.


      import { NgModule, APP_INITIALIZER } from '@angular/core';
      import { ConfigService } from '../services/config.service';


NgModule부터 보시면 providers에 APP_INITIALIZER와 ConfigService 그리고 getConfig라는 factory를 설정하였습니다. multi값도 true로 설정하였습니다.
여기에서 바로 제공되는 시점, 사용할 service명, 어떠한 factory를 활용할 지를 정의하는 것입니다.

      {
            provide: APP_INITIALIZER,
            useFactory: getConfig,
            deps: [ConfigService],
            multi: true
      }


이제 module 코드 중간을 보시면 getConfig라는 함수를 구현하였는데 이게 factory에 정의된 함수 입니다.
내용은 단순히 configService의 get 함수를 호출하는 것입니다.

      export function getConfig(configService: ConfigService) {
            return () => configService.get();
      }


이제 모두 설정되었습니다. 코드를 조금 더 수정하여 component에서 값을 사용할 수 있도록 수정한다면 component 시작 시점에 이미 서비스에서 설정한 값을 사용할 수 있습니다.

끝.