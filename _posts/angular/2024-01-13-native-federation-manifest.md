---
title: "Micro Frontend with Native Federation by Manifest"
date: 2024-01-13 14:11:00 +0900
comments: true
categories: angular
tags: [nativefederation, manifest]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Angular-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EC%97%90-Native-Federation%EC%9D%98-manifest%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Micro-Frontend-%EA%B5%AC%EC%B6%95)
<br/>

In this article, we will explore the process of building a Micro Frontend using Native Federation, with practical examples.

## Native Federation Definition

Native Federation is a library that supports building Micro Frontend environments using various bundlers such as esbuild, leveraging module federation.

In Angular, module federation is used to construct Micro Frontend environments. [Related Link](https://www.npmjs.com/package/@angular-architects/module-federation). Currently, module federation supports Angular versions 12 to 16, and can be used in webpack environments beyond those versions.

However, if you are using Angular 16 or higher and employing the esbuild bundler, utilizing module federation may be challenging. Therefore, we recommend constructing a micro frontend environment using Native Federation. [Related Link](https://www.npmjs.com/package/@angular-architects/native-federation)

## Overview

Micro Frontend fundamentally involves connecting two or more projects. Therefore, in this article, we will create two projects: one as the host and the other as a remote project. We will then write code that allows the host to import components from the remote project.

## Host (Shell) Project Setup

### Project Creation

First, create the host project. In this example, we are creating a project based on Angular 17 with esbuild, and naming the project `native-federation-shell`.

```

### Native Federation Installation

```

### Native Federation Configuration

Set the `type` to `dynamic-host` to accommodate various scenarios.

```

Once the configuration is complete, you can see that the `federation.config.js` file has been created and the settings in `angular.json` have been modified.

Also, you can verify that `es-module-shims` has been added to the `dependencies` in `package.json` (^1.5.12 or higher).

### Manifest Configuration

Create a `federation.manifest.json` file in the `assets` directory. Here, the key is the remote project name, and the value is the URL of the remote project.

```

{
	"native-federation-remote-manifest": "http://localhost:4201/remoteEntry.json"
}

```

### main.ts Configuration

`initFederation` must be called before calling the `bootstrap.ts` file. Additionally, connect the `manifest` file as a parameter to `initFederation`.

```

import { initFederation } from '@angular-architects/native-federation';

initFederation('/assets/federation.manifest.json')
  .catch(err => console.error(err))
  .then(_ => import('./bootstrap'))
  .catch(err => console.error(err));

```

### Calling a Remote Component from a Route

In this article, we will call a component from the remote project in a route. At this time, you must verify that the component has been exposed in the remote project's `federation.config.js`.

In this example, we are connecting two remote components.

```

// app.routes.ts
import { loadRemoteModule } from '@angular-architects/native-federation';
import { Routes } from '@angular/router';

export const routes: Routes = [
  {path: 'remote-app', loadComponent: () =>
    loadRemoteModule('native-federation-remote-manifest', './Component').then((m) => m.AppComponent),
  },
  {path: 'remote-sample', loadComponent: () =>
    loadRemoteModule('native-federation-remote-manifest', './SampleComponent').then((m) => m.SampleComponent),
  }
];

```

#### Calling a Remote Route from a Route

You can also call a remote route. In this case, you should use `loadChildren` instead of `loadComponent`.

```

{
    path: 'remote-routes',
    // loadChildren instead of loadComponent !!!
    loadChildren: () =>
      loadRemoteModule('native-federation-remote-manifest', './routes').then((m) => m.APP_ROUTES),
  },

```

### Template Creation

Now, call the route using an `<a>` tag.

```

<a [routerLink]="['/remote-app']" routerLinkActive="router-link-active" >REMOTEAPP</a>
<br/>
<br/>
<a [routerLink]="['/remote-sample']" routerLinkActive="router-link-active" >REMOTESAMPLE</a>
<br/>
<br/>

<router-outlet></router-outlet>

```

The host setup is now complete.

## Remote Project Setup

Now, create the remote project and connect it to the host.

### Project Creation

```

### Native Federation Installation

```

### Native Federation Configuration

Unlike the host, set the `type` to `remote`.

```

Once the configuration is complete, `federation.config.js`, `angular.json`, and `package.json` will be added or modified, similar to the host.

### federation.config.js Modification

Similar to the host's configuration, but in the remote, you must write the components to expose to the host. Here, we define the two components defined in the host's route. Also, write the same name as the host manifest.

```jsx
const { withNativeFederation, shareAll } = require('@angular-architects/native-federation/config');

module.exports = withNativeFederation({

  name: 'native-federation-remote-manifest',

  exposes: {
    './Component': './src/app/app.component.ts',
    './SampleComponent': './src/app/sample/sample.component.ts',
  },

  shared: {
    ...shareAll({ singleton: true, strictVersion: true, requiredVersion: 'auto' }),
  },

  skip: [
    'rxjs/ajax',
    'rxjs/fetch',
    'rxjs/testing',
    'rxjs/webSocket',
    // Add further packages you don't need at runtime
  ]
  
});
```

### main.ts Creation

Since the remote does not have a manifest, define `initFederation` with no parameters.

```tsx
import { initFederation } from '@angular-architects/native-federation';

initFederation()
  .catch(err => console.error(err))
  .then(_ => import('./bootstrap'))
  .catch(err => console.error(err));
```

### sample.component.ts Creation

Create a component to expose arbitrarily.

### .ts Error Case

If a TS file error occurs when trying to expose a component other than `appComponent`, include the TS file in `tsconfig.app.json`.

```json
{
  ...
  "include": [
    "src/**/*.d.ts",
    "src/**/*.ts",
  ]
}
```

## Execution

Run both the host and remote projects, and click the link on the host to verify that the remote component is being called correctly.

## Conclusion

We have built a micro frontend where the host uses the remote's component as if they were a single project, using native-federation. This confirmed that a single project can be built through several different projects, and also confirmed that native federation allows building a micro frontend environment in projects that use the esbuild bundler instead of webpack.

The method is similar to module federation, with only some differences in settings, so please refer to it. See the reference sites below for more detailed and accurate information.

## Reference Sites

[micro-frontends-with-modern-angular-part-1-standalone-and-esbuild](https://www.angulararchitects.io/blog/micro-frontends-with-modern-angular-part-1-standalone-and-esbuild/)