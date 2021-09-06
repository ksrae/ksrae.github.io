---
title: "Angular Universal 프로젝트를 Github에서 Heroku로 배포 (Deploy Angular Universal Project from GitHub to Heroku)"
date: 2019-07-29 12:26:00 +0900
comments: true
categories: angular
tags: [heroku, deploy]
---



 
 github에 업로드 된 Angular Universal 프로젝트를 Heroku에 업로드 하는 방법을 알아보겠습니다.<br><br>

사실 업로드 방법이 어렵지 않으나 헤맨 부분의 설명이 명확히 없어서 정리해보려고 합니다.


## 프로젝트 생성

- 먼저 heroku에 가입한 뒤,
- new를 클릭하여 프로젝트를 생성합니다.
- 프로젝트를 선택합니다.



## Dyno formation 확인
 
- Overview 탭에서 Dyno formation을 확인합니다. 무료 버전에서는 npm start가 고정되어 있습니다.


```
This app is using free dynos

web npm start ON
```




## Deploy 설정

- Deploy 탭으로 이동합니다.
- Deployment Method에서 Github를 선택합니다.
- App connected to Github에서 배포할 프로젝트를 선택합니다.
- Automatic deploys 또는 Manual deploy 중 하나를 선택합니다. git에 익숙하면 Manual. 그렇지 않으면 Automatic에서 원하는 branch를 선택하시면 됩니다. 저는 Automatic을 선택했습니다.






## package.json 수정


무료 버전에서는 Dyno 수정이 어려우므로 기본 package.json의 scripts를 수정합니다.<br>
Dyno 수정이 가능하면 실행 명령을 직접 Dyno에 기입하시면 되므로 이 과정이 필요 없습니다.

기본 package.json
```json
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e",
    "compile:server": "webpack --config webpack.server.config.js --progress --colors",
    "serve:ssr": "node dist/server",
    "build:ssr": "npm run build:client-and-server-bundles && npm run compile:server",
    "build:client-and-server-bundles": "ng build --prod && ng run ssr01:server:production --bundleDependencies all"
  },
```

### start 수정
start 를 heroku에서 server.js 를 실행 가능하도록 수정합니다. 일반적으로 Angular Universal 프로젝트는 빌드 후 dist 폴더에 server.js가 있으므로 dist/server로 설정합니다.

```json
"start": "node dist/server"
```


### heroku-postbuild 추가
heroku-postbuild 항목을 추가합니다. heroku 에서는 빌드 시도 시 heroku-postbuild와 build를 체크합니다. 따라서 build를 수정해도 되지만 여기에서는 heroku-postbuild를 추가하고 build를 제거하겠습니다.

```json
<del>"build": "ng build",</del>
"heroku-postbuild": "npm run build:ssr"
```

간단히 angular universal build 명령을 실행하도록 했습니다.


### deploy 추가 (optional)

제 프로젝트에서는 필요 없었으나 일부 사이트에서 적용 방법을 설명하고 있기에 정리해 보았습니다. 아마도 deploy 명령을 manual 하게 적용할 때 사용할 수 있는 것 같습니다. 


```json
"deploy": "npm run build:ssr && git subtree push --prefix dist heroku master",
```


### 변경된 package.json
```json
  "scripts": {
    "ng": "ng",
    "start": "node dist/server",
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e",
    "compile:server": "webpack --config webpack.server.config.js --progress --colors",
    "serve:ssr": "node dist/server",
    "build:ssr": "npm run build:client-and-server-bundles && npm run compile:server",
    "build:client-and-server-bundles": "ng build --prod && ng run searchword:server:production --bundleDependencies all",
    "deploy": "npm run build:ssr && git subtree push --prefix dist heroku master",
    "heroku-postbuild": "npm run build:ssr"
  },
```

## Upload / build

Automatic인 경우 선택한 브랜치에 프로젝트가 업로드 될 때, <br/>
Manual 인 경우 Deploy 버튼을 클릭 할 때, <br/>
빌드가 시작됩니다. 빌드 진행 과정은 deploy 탭이나 overview 탭에서 확인할 수 있습니다.


## 실행

우측 상단에 Open App 버튼을 클릭하여 확인합니다.


## Log 또는 cli 실행

에러가 발생한 경우 Log를 확인하거나 콘솔에서 직접 명령을 실행하는 방법을 확인해보겠습니다.
- 우측 상단의 More를 클릭합니다.
> View Log로 Log를 확인하거나
> Run Console로 실행을 테스트 해봅니다.





## 참고 사이트
- [How to Deploy Angular Application to Heroku - Olutunmbi Banto - Medium](https://medium.com/@hellotunmbi/how-to-deploy-angular-application-to-heroku-1d56e09c5147)