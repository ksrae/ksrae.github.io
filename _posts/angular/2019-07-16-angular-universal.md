---
title: "Angular Universal"
date: 2019-07-16 17:14:00 +0900
comments: true
categories: angular
tags: [ssr, seo, universal]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Angular-Universal)
<br/>

# Implementing Server-Side Rendering (SSR) in Angular

Angular, being a robust framework, provides a straightforward approach to implement Server-Side Rendering (SSR). This guide outlines the steps to integrate SSR into your Angular application, enabling improved performance and SEO.

## How SSR Works

The Universal web server renders static HTML pages using a template engine. This allows the server to process the Document Object Model (DOM), XMLHttpRequest, and low-level operations independently, without relying on a browser.

<br>

The server forwards client requests to the `ngExpressEngine` and leverages the `renderModuleFactory()` function to render the content within template tags before delivering it to the client. This pre-rendering enhances the initial loading speed and provides better support for search engine crawlers.

## Supported Engines

Angular Universal supports several server-side rendering engines:

```
@nguniversal/express-engine
@nguniversal/aspnetcore-engine
@nguniversal/hapi-engine
@nguniversal/socket-engine
```

Additionally, engines based on Mozilla's `dom.js` and a common engine for shared processing are under development and continuously updated. The selection of the appropriate engine depends on your project's server-side environment.

## Project Setup

Start by creating a new Angular project. The process is identical to creating a standard Angular project.

```bash
ng new ssr_project
```

This command generates a project named `ssr_project`.

## Adding the Universal Engine

Navigate to the project directory and add the server engine. This guide uses the Express engine for Node.js. Ensure Node.js is installed.

```bash
ng add @nguniversal/express-engine --clientProject ssr_project
```

This command adds necessary configuration files and dependencies to your project, facilitating server-side rendering capabilities.

<br>

After running this command, several files are added to the project. These files are essential for server-side configuration and Angular bootstrapping. Let's examine each file:

## File Analysis

### server.ts

This file contains the code to start the Node.js server and invokes the Express engine.

```tsx
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

When configuring the server, differentiate between page calls within the application, API calls, and asset calls.

<br>

This can be achieved by structuring your application as follows:

- Configure API calls within the `/api` route, ensuring that `/api` calls are treated as data requests.
- Treat calls without a file extension as page calls within the application.
- Consider other calls as asset requests.

If you structure your application like this, Node.js can distinguish between links as follows:

```tsx
app.get('/api/*', (req, res) => {
  res.status(404).send('data requests are not supported');
});

app.get('*', (req, res) => {
  res.render('index', { req });
});

app.get('*.*', express.static(join(DIST_FOLDER, 'browser')));
```

### tsconfig.server.json

While the existing `tsconfig.app.json` file configures the client-side, `tsconfig.server.json` configures the server-side.

<br>

The configuration is not significantly different; however, the `angularCompilerOptions` section defines the `entryModule`.

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

This file contains the Webpack configuration for the server.

```jsx
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

This file contains the code to bootstrap the application on the server.

```tsx
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

This module is called from the server-side.

```tsx
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

The SSR setup is now complete. Angular simplifies the SSR setup with just a few commands.

## Important Considerations

- Since rendering occurs on the server rather than the client, browser commands are no longer available.
- `window`, `document`, `navigator`, and `location` are no longer accessible directly.
- To address this, Angular provides injectable APIs such as `Location` and `DOCUMENT`.

(More precisely, they are unavailable only when the platform's state is `server`. For a detailed explanation, refer to [Angular Universal Platform](https://ksrae.github.io/angular/universal-platform).)

- Similarly, clicks and user events cannot be processed server-side. You must determine the rendering target for these events and implement routing functionality.
- Links must use absolute URL values. The official Angular documentation suggests the following code as a guideline.

Create `universal-interceptor.ts`.

```tsx
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

Add the created file to `app.server.module.ts`.

```tsx
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

Now, all HTTP requests performed on the server will be changed to absolute URLs by this interceptor.

## Execution

Execute the commands defined in the `scripts` section of `package.json`.

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

### Build

Run the `build:ssr` command.

```bash
npm run build:ssr
```

### Start the Server

Execute the `serve:ssr` command in `package.json`, or navigate to the `dist` folder and run the `node server` command.

```bash
npm run serve:ssr
```

Or

```bash
dist> node server
```

### Verify in Browser

After starting the server, check the port set in `server.ts` (4000 in this case).

<br>

If running locally, check `localhost:4000`.

<br>

If the page looks the same as the original Angular project, the setup is successful.

<br>

## Resources

- [Angular Universal Guide](https://angular.io/guide/universal)