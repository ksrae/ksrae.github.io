---
title: "angular project on electron"
date: 2019-09-16 10:19:00 +0900
comments: true
categories: angular
tags: [electron, error]
---


- electron 설치
electron을 npm에서 간단히 받아 설치할 수 있습니다.

```
npm i electron@latest
```


- angular 프로젝트 빌드
angular project를 electron에서 실행하려면 우선 프로젝트를 빌드하여 dist 폴더를 생성합니다.

```
ng build
```

- main.js 생성
electron은 js 파일을 수행하여 html을 load하는 방식이므로 js 파일을 생성하여야 합니다.
아래와 같이 main.js 파일을 생성합니다.

```js
const {app, BrowserWindow} = require('electron')
const url = require("url");
const path = require("path");

let mainWindow

function createWindow () {
  mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true
    }
  })

  mainWindow.loadURL(/*url 또는 파일명*/)


  mainWindow.on('closed', function () {
    mainWindow = null
  })
}

app.on('ready', createWindow)

app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
})

app.on('activate', function () {
  if (mainWindow === null) createWindow()
})

```

- 프로젝트 load

angular 프로젝트를 local이나 서버에 올린 뒤 주소를 electron에서 로드하거나 파일을 직접 로드할 수 있습니다.
loadURL에서 이를 설정할 수 있습니다. 만일 파일인 경우 정확한 경로와 실행할 html 파일을 설정하여야 합니다.

```js

  // online에서 실행
  mainWindow.loadURL('http://127.0.0.1:4000')

  // offline에서 실행
  mainWindow.loadURL(
    url.format({
      pathname: path.join(__dirname, `/dist/index.html`),
      protocol: "file:",
      slashes: true
    })
  );

```


- package.json 수정

package.json을 설정을 수정하여 쉬운 실행을 설정합니다.

```
  "scripts": {
	...
	"start:electron": "ng build --prod --base-href ./ && electron .",
    "electron": "electron .",
  }
  ```

angular 빌드와 electron을 한번에 실행하기 위한 start:electron과 electron만 별도로 실행할 두개의 스크립트를 생성했습니다.
하나만 생성해도 상관 없습니다. 또한 prod 와 base-href 설정을 추가하였는데 빼도 상관 없습니다.

이제 스크립트를 실행하면 앱으로 실행되는 것을 확인할 수 있습니다.


- 에러 해결

> 위의 설정대로 진행하였어도 실행 안되면 빌드 후 생성된 index.html에서 두 가지를 확인하시기 바랍니다.
> 1) base href 가 "./" 으로 되어 있는가? 
> 기본 "/" 으로 설정되어 있으므로 "./" 으로 수정.
>
> 2) script의 type이 text/javascript 인가
> 기본 type="module"로 되어 있으므로 "text/javascript"로 수정.



- 참고 사이트
https://www.sitepoint.com/build-a-desktop-application-with-electron-and-angular/
[Build Angular Desktop Apps With Electron \| AngularFirebase](https://angularfirebase.com/lessons/desktop-apps-with-electron-and-angular/)
