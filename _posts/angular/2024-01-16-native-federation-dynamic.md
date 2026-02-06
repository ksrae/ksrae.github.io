---
title: "Micro Frontend with Native Federation without Manifest"
date: 2024-01-16 13:06:00 +0900
comments: true
categories: angular
tags: [nativefederation, dynamic]
---

## Building Micro Frontends Dynamically with Native Federation: An Example

In a [previous article](https://ksrae.github.io/angular/native-federation-manifest/), we explored building Micro Frontends using a static approach, defining remotes using a manifest. This article demonstrates how to construct Micro Frontends in Native Federation by dynamically defining remotes without relying on a manifest. We'll illustrate this with a practical example.

## Overview

Since the remote configuration remains largely unchanged from the previous article, we'll focus primarily on the host (shell) implementation in this guide.

## Host (Shell) Project Setup

### Project Creation

First, we'll create the host project. For this example, we'll use Angular 17 with esbuild. The project will be named `native-federation-shell`.

```bash
ng new native-federation-shell
```

### Native Federation Installation

Install the Native Federation library as a development dependency:

```bash
npm i @angular-architects/native-federation -D
```

### Native Federation Configuration

Configure Native Federation to support dynamic remote loading by setting the `type` to `dynamic-host`.

```bash
ng g @angular-architects/native-federation:init --type dynamic-host
```

After running this command, a `federation.config.js` file will be created, and the `angular.json` configuration file will be modified. Additionally, `es-module-shims` will be added as a dependency in `package.json` (version ^1.5.12 or higher).

### `main.ts` Configuration

Before calling `bootstrap.ts`, it's essential to invoke `initFederation`.  Since we're not defining a manifest, `initFederation` is used without any parameters, similar to how it's used with remotes.

```tsx
import { initFederation } from '@angular-architects/native-federation';

initFederation()
  .catch(err => console.error(err))
  .then(_ => import('./bootstrap'))
  .catch(err => console.error(err));
```

### Defining `loadRemoteModule`

Implement the logic to dynamically load components from remotes within a component.  While we use the same `loadRemoteModule` function as when loading remote components in the router, it's important to note the difference in parameters.

```tsx
function loadRemoteModule<T = any>(options: LoadRemoteModuleOptions): Promise<T>;

export type LoadRemoteModuleOptions = {
    remoteEntry?: string; // url
    remoteName?: string; // Must match the name defined in the remote's federation.config.js
    exposedModule: string; // Key value of the exported component
};
```

### Defining `federation.config.js` in the Remote

Based on the above types, create the code to call the remote component. The remote URL is http://localhost:4202, and the `federation.config.js` is configured as follows:

```jsx
const { withNativeFederation, shareAll } = require('@angular-architects/native-federation/config');

module.exports = withNativeFederation({
  // Must match the remoteName defined in the host.
  name: 'native-federation-remote',

  // The value must be included in the key value defined in exposes when setting exposedModule in the host.
  exposes: {
    './DynamicComponent': './src/app/dynamic/dynamic.component.ts',
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
  ],

});
```

### Defining a DOM to Replace `router-outlet`

When calling a remote component from a component (not through routing), you cannot use `router-outlet`.  Therefore, you must define a DOM element to take its place.  To add the remote component to the defined DOM, you need the `createComponent()` function, which can be obtained from `ViewContainerRef`.  To get `ViewContainerRef`, the DOM must be imported as a `ViewChild` and defined as `ViewContainerRef`.

```tsx
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [],
  templateUrl: '<div #remote></div>',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class AppComponent {
  title = 'native-federation-shell';
  cdr = inject(ChangeDetectorRef);

	// Because it is the location to import the Remote component, import the DOM information as ViewChild. read and static settings are required.
  @ViewChild('remote', {read: ViewContainerRef, static: true}) viewContainer!: ViewContainerRef;

  ...
}
```

### Calling the Remote Component

Finally, use `loadRemoteModule()` to get the remote component and then add it to the defined DOM with `createComponent()`. If the component's `changeDetection` is set to `OnPush`, you must manually define `ChangeDetectorRef`.

```tsx
import { ChangeDetectionStrategy, ChangeDetectorRef, Component, ViewChild, ViewContainerRef, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { loadRemoteModule } from '@angular-architects/native-federation';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule],
  templateUrl: '<div #remote></div>',
  styleUrl: './app.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class AppComponent {
  title = 'native-federation-shell';
  cdr = inject(ChangeDetectorRef);

  @ViewChild('remote', {read: ViewContainerRef, static: true}) viewContainer!: ViewContainerRef;

  async openDynamic(e: any) {
    const remoteComponent = await loadRemoteModule({
			// Required if there is no manifest. In module federation, it was remoteEntry.js, but it has been changed to .json.
      remoteEntry: 'http://localhost:4202/remoteEntry.json',
      exposedModule: './DynamicComponent',
      remoteName: 'native-federation-remote',
    }).then(m => m.DynamicComponent); // component to be exposed defined in remote

    const ref = this.viewContainer.createComponent(remoteComponent);
    this.cdr.markForCheck(); // Must be defined if changeDetection is OnPush.
  }
}
```

## Conclusion

When Native Federation is configured dynamically, the relationship between the host and remote becomes less rigid. Any project can be defined as a host or a remote, and even simultaneously as both.

In essence, if a component is defined in `expose`, it becomes possible to construct projects by reconfiguring components from various projects, offering a wide range of project structuring possibilities.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EC%97%90-Native-Federation%EC%97%90%EC%84%9C-%EB%8F%99%EC%A0%81%EC%9C%BC%EB%A1%9C-Micro-Frontend-%EA%B5%AC%EC%B6%95)
<br/>