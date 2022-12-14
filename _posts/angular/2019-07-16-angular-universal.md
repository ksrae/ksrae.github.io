---
title: "Angular Universal"
date: 2019-07-16 17:14:00 +0900
comments: true
categories: angular
tags: [ssr, seo, universal]
---



Angular에서 SSR을 사용하는 방법입니다. 우수한 Framework 답게 쉽고 빠르게 적용할 수 있도록 잘 구성되어 있기 때문에, 아래의 예제만 따라해도 충분히 실전에 사용할 수 있을 것 입니다.


## 동작 원리 

Universal 웹 서버는 static HTML 페이지를 template 엔진에 랜더링 하며, 브라우저의 도움 없이 DOM, XMLHttpRequest 또는 low-level 을 서버에서 처리할 수 있습니다.<br><br>
서버는 클라이언트의 요청을 ngExpressEngine에 전달하고 `renderModuleFactory()` 함수를 통해 template 태그의 내용을 랜더링하여 client에 전달합니다.


## 지원 여부
Angular Universal은 대표적으로 4가지 방식의 엔진을 지원하고 있습니다. 

    @nguniversal/express-engine
    @nguniversal/aspnetcore-engine
    @nguniversal/hapi-engine
    @nguniversal/socket-engine
    

이 외에도 모질라의 dom.js 기반  및  공통으로 처리할 common 등이 개발되어 있으며 계속 업데이트 중에 있습니다.


## 프로젝트 생성

angular 프로젝트를 생성합니다. 기존 angular 프로젝트 생성과 방식은 동일합니다.

```bash
ng new ssr_project
```

ssr_project라는 이름의 프로젝트를 생성하였습니다.


## universal 엔진 추가

프로젝트가 생성된 폴더로 이동 후 서버 엔진을 추가합니다. 여기에서는 Node.js의 express 엔진을 사용하겠습니다. 따라서 Node가 반드시 설치 되어 있어야 합니다.

```bash
ng add @nguniversal/express-engine --clientProject ssr_project
```


이 명령이 완료되면 폴더에 몇 개의 파일이 추가되어 있을 것입니다.<br>
이는 서버에서의 설정 및 Angular boostrap 등에 필요한 파일입니다.<br>
하나씩 알아보겠습니다.

## 분석

### server.ts

노드 서버를 가동하는 코드 이며, express 엔진을 호출하고 있습니다.

```ts
import 'zone.js/dist/zone-node';

import * as express from 'express';
import {join} from 'path';

const app = express();

const PORT = process.env.PORT || 4000;
const DIST_FOLDER = join(process.cwd(), 'dist/browser');

const {AppServerModuleNgFactory, LAZY_MODULE_MAP, ngExpressEngine, provideModuleMap} = require('./dist/server/main');

app.engine('html', ngExpressEngine({
  bootstrap: AppServerModuleNgFactory,
  providers: [
    provideModuleMap(LAZY_MODULE_MAP)
  ]
}));

app.set('view engine', 'html');
app.set('views', DIST_FOLDER);

app.get('*.*', express.static(DIST_FOLDER, {
  maxAge: '1y'
}));

app.get('*', (req, res) => {
  res.render('index', { req });
});

app.listen(PORT, () => {
  console.log(`Node Express server listening on http://localhost:${PORT}`);
});

```


서버를 구성할 때 유의할 점은 app 내의 페이지 호출과 다른 api 호출, asset 호출을 구분하는 것입니다.<br>
이는 아래와 같이 정의하여 구분할 수 있습니다.

- api 호출은 /api 내부에 구성하여 /api 호출은 data 호출로
- /api 호출이 아니고 파일 확장자가 없으면 app 내의 페이지 호출로
- 기타 호출은 asset 호출로


만일 이와 같이 구성한 경우 node 에서는 다음과 같이 link를 구분하여 처리할 수 있습니다.

```ts
app.get('/api/*', (req, res) => {
  res.status(404).send('data requests are not supported');
});

app.get('*', (req, res) => {
  res.render('index', { req });
});

app.get('*.*', express.static(join(DIST_FOLDER, 'browser')));
```



### tsconfig.server.json

기존 tsconfig.app.json 파일은 클라이언트의 설정이라면 tsconfig.server.json은 서버의 설정입니다.<br><br>
구성은 크게 다르지 않으나 `angularCompilerOptions`에서 entryModule을 정의하고 있는 점이 특징입니다.

```json
{
  "extends": "./tsconfig.app.json",
  "compilerOptions": {
    "outDir": "./out-tsc/app-server"
  },
  "angularCompilerOptions": {
    "entryModule": "./src/app/app.server.module#AppServerModule"
  }
}

```


### webpack.server.config.js

webpack의 서버 설정 입니다.

```js
const path = require('path');
const webpack = require('webpack');

module.exports = {
  mode: 'none',
  entry: {
    server: './server.ts'
  },
  externals: {
    './dist/server/main': 'require("./server/main")'
  },
  target: 'node',
  resolve: { extensions: ['.ts', '.js'] },
  optimization: {
    minimize: false
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].js'
  },
  module: {
    noParse: /polyfills-.*\.js/,
    rules: [
      { test: /\.ts$/, loader: 'ts-loader' },
      {
        test: /(\\|\/)@angular(\\|\/)core(\\|\/).+\.js$/,
        parser: { system: true },
      },
    ]
  },
  plugins: [
    new webpack.ContextReplacementPlugin(
      /(.+)?angular(\\|\/)core(.+)?/,
      path.join(__dirname, 'src'), // location of your src
      {} // a map of your routes
    ),
    new webpack.ContextReplacementPlugin(
      /(.+)?express(\\|\/)(.+)?/,
      path.join(__dirname, 'src'),
      {}
    )
  ]
};

```

### src/main.server.ts

서버에서 boostrap 하기 위한 코드 입니다.

```ts
import { enableProdMode } from '@angular/core';

import { environment } from './environments/environment';

if (environment.production) {
  enableProdMode();
}

export { AppServerModule } from './app/app.server.module';
export { ngExpressEngine } from "@nguniversal/express-engine";
export { provideModuleMap } from "@nguniversal/module-map-ngfactory-loader";

```




### src/app/app.server.module.ts

서버에서 호출하는 서버사이드용 module 입니다.

```ts
import { NgModule } from '@angular/core';
import { ServerModule } from '@angular/platform-server';

import { AppModule } from './app.module';
import { AppComponent } from './app.component';
import { ModuleMapLoaderModule } from '@nguniversal/module-map-ngfactory-loader';

@NgModule({
  imports: [
    AppModule,
    ServerModule,
    ModuleMapLoaderModule,
  ],
  bootstrap: [AppComponent],
})
export class AppServerModule {}

```


이제 SSR 설정을 마쳤습니다. Angular에서는 이렇게 명령어 한 두개로 쉽게 SSR 설정을 완료할 수 있습니다.



## 유의점

- 랜더링 시점이 클라이언트가 아닌 서버로 변경되었으므로, 브라우저 명령을 더 이상 사용할 수 없습니다.
- 즉, window, document, navigator, location은 더 이상 사용할 수 없습니다.
- 이를 해결하기 위해 Angular에서는 Injectable한 Location, DOCUMENT와 같은 몇 가지 API를 제공합니다.
(더 정확히는 platform의 상태가 server 일 때만 사용 불가 합니다. 더 자세한 설명은 [Angular Universal Platform](https://ksrae.github.io/angular/universal-platform) 을 확인하시기 바랍니다.)


- 마찬가지로 클릭이나 유저의 이벤트 또한 서버 사이드에서는 처리할 수 없으므로 이를 처리할 랜더링 대상을 결정해야 하며, 반드시 Routing 기능을 구현하여야 합니다.


- 링크는 반드시 절대 링크 값을 사용해야 합니다. Angular 공식 사이트에서는 아래의 코드를 가이드로 제시하고 있습니다.


universal-interceptor.ts 를 생성합니다.

```ts
import {Injectable, Inject, Optional} from '@angular/core';
import {HttpInterceptor, HttpHandler, HttpRequest, HttpHeaders} from '@angular/common/http';
import {Request} from 'express';
import {REQUEST} from '@nguniversal/express-engine/tokens';
 
@Injectable()
export class UniversalInterceptor implements HttpInterceptor {
 
  constructor(@Optional() @Inject(REQUEST) protected request: Request) {}
 
  intercept(req: HttpRequest, next: HttpHandler) {
    let serverReq: HttpRequest = req;
    if (this.request) {
      let newUrl = `${this.request.protocol}://${this.request.get('host')}`;
      if (!req.url.startsWith('/')) {
        newUrl += '/';
      }
      newUrl += req.url;
      serverReq = req.clone({url: newUrl});
    }
    return next.handle(serverReq);
  }
}

```

작성한 파일을 app.server.module.ts에 추가합니다.

```ts
import {HTTP_INTERCEPTORS} from '@angular/common/http';
import {UniversalInterceptor} from './universal-interceptor';
 
@NgModule({
  ...
  providers: [{
    provide: HTTP_INTERCEPTORS,
    useClass: UniversalInterceptor,
    multi: true
  }],
})
export class AppServerModule {}
```

이제 서버에서 수행된 모든 http 요청은 이 인터셉터에 의해 절대 url로 변경됩니다.




## 실행

package.json을 열어 scripts에 정의되어 있는대로 실행하면 됩니다.

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

### 빌드

build:ssr 명령을 실행합니다.

```bash
npm run build:ssr
```


### 서버 기동

package.json의 serve:ssr 명령을 수행하거나 dist 폴더로 이동하여 node server 명령을 수행해도 됩니다.

```bash
npm run serve:ssr
```

또는 

```bash
dist> node server
```

### 브라우저에서 확인

서버 가동 후 server.ts 에 설정된 포트 (여기에서는 4000) 를 확인합니다.<br>
로컬로 실행할 경우 localhost:4000을 확인합니다.<br><br>


페이지가 기존 Angular 프로젝트와 동일하게 보인다면 성공입니다.<br><br>



## 참고 사이트
- [Angular](https://angular.io/guide/universal)
