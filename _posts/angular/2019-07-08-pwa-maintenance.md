---
title: "PWA로 오프라인 공지하기 (Make Matenenance or Offline Notice using PWA)"
date: 2019-07-08 16:50:00 +0900
comments: true
categories: angular
tags: [pwa, maintenance, offline]
---



PWA를 활용하여 점검 또는 오프라인 공지를 만드는 방법입니다.




## 목적
  - 쉽게 점검 표시를 하기 위한 방법을 PWA에 적용하기
  - offline일 때 앱은 살아 있되 사용자가 offline임을 인지할 수 있도록 함


## 점검하기
가장 빠른 점검 메시지를 적용하기 위해 .json 에 기록 후 읽어가는 방식을 활용합니다.


## 문제
PWA로 개발 시 .json 파일이 cache에 물려있게 되며 이 때, 두 가지 문제가 발생합니다.

- 접속 시 .json의 업데이트 적용에 간격이 있고, 그 동안 사용자를 대기 시킬 수 없습니다.
- offline 시에도 캐싱된 .json을 읽어버리기 때문에 별도의 메시지를 표시하기 어렵습니다.


## 해결
ngsw.json 에서 .json 파일 내역을 지웁니다. 즉, 캐싱되지 않도록 합니다.

- 최초로 load되는 module에서 json 파일을 읽기.
  - 성공: 결과를 변수 또는 subject에 저장.
  - 실패: catch 이벤트에서 처리.

- component는 onInit 시 해당 변수 또는 해당 subject를 subscribe하여 결과를 반영합니다.

## 결과

- maintenance가 업데이트와 관계없이 즉시 적용됩니다.
- offline 안내 및 재접속 또는 네트워크 확인 안내 등이 가능합니다.


## 소스 코드

```json
// maintenance.json
{
    "maintenance": false
}
```

```json
// ngsw.json (maintenance.json 파일이 캐싱되지 않음)

{
  "configVersion": 1,
  "index": "/index.html",
  "assetGroups": [
    ...
    {
      "name": "assets",
      "installMode": "lazy",
      "updateMode": "prefetch",
      "urls": [
        "/assets/icons/icon-128x128.png",
      ],
      "patterns": []
    }
  ],
  "dataGroups": [],
  "hashTable": {
    "/assets/icons/icon-128x128.png": "dae3b6ed49bdaf4327b92531d4b5b4a5d30c7532",
    ...
  },
  "navigationUrls": [
    ...
  ]
}
```

```ts
// app.module.ts

export function getConfig(config: AppService) {
  return async () => await config.getMaintenance();
}
...
  providers: [
    HttpClient,
    {
      provide: APP_INITIALIZER,
      useFactory: getConfig,
      deps: [AppService],
      multi: true
    }
  ]
```

```ts
// app.service.ts

    getMaintenance() {
        return new Promise((resolve, reject) => {
            this.http.get(`${window.location.origin}/assets/maintenance.json`, { headers: new HttpHeaders({ timeout: `${3000}`})})
            .toPromise().then(item => {
                this.config = item['maintenance'];
                resolve();
            }).catch(err => {
                this.offline = true;
                resolve();
            });
        });
    }
```

```ts
// app.component.ts

  ngOnInit() {
    const maintenance = this.appService.config;
    const offline = this.appService.offline;
    if (maintenance) {
      alert('MAINTENANCE');
    }
    if (offline) {
      alert('offline');
    }
    
    if (this.swUpdate.isEnabled) {
      ...
    }
```


## 응용

이제 점검 중일 때는 maintenance.json의 "maintenance" 값을 true로 수정한 뒤 이 파일만 배포하면 되며, <br>
offline일 시 오프라인임을 알려주거나 네트워크 점검이 필요하다는 안내문구를 넣을 수 있습니다.<br><br>

"maintenance"에 시간 값을 추가하여 client에서 해당 시간 범위만큼일 때만 maintenance의 값을 처리하게 할 수도 있을 것 입니다.<br>




