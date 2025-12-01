---
title: "Creating Web Component in Angular: Standalone Based"
date: 2024-01-01 14:46:00 +0900
comments: true
categories: angular
tags: [standalone, webcomponent]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Angular-Standalone-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-Web-Component-%EB%A7%8C%EB%93%A4%EA%B8%B0)
<br/>

# Creating Web Components in Angular Standalone Environments: A Comprehensive Guide

This article explores the process of creating Web Components within Angular. While numerous resources focus on Module-based approaches, this guide specifically addresses the creation of Web Components in Angular Standalone environments. For information on Module-based implementations, please refer to alternative documentation.

## Workspace Configuration

To facilitate organized management, we'll begin by establishing a workspace:

```bash
ng new workspace --no-create-application
```

Next, we'll add both a Library, serving as the source of our Web Component, and an Application to encapsulate and transform it into a Web Component:

```bash
ng g library web-component-lib
```

```bash
ng g application web-component-app
```

## Library Implementation

Within the library code, implement your desired functionality. You can leverage Angular features such as `@Input` and `@Output` as needed.

Optionally, configure the Shadow DOM for encapsulation.

```tsx
import { Component, EventEmitter, Input, Output, ViewEncapsulation } from '@angular/core';

@Component({
  selector: 'lib-web-component-lib',
  standalone: true,
  imports: [],
  template: `
    {{ setValue }}
    <button (click)="onClick($event)">Emit</button>
  `,
  styles: ``,
  encapsulation: ViewEncapsulation.ShadowDom // optional
})
export class WebComponentLibComponent {
  @Input() setValue!: string;
  @Output() onResult = new EventEmitter();

  onClick(e: any) {
    this.onResult.emit(e);
  }
}
```

## Application Implementation

Since our objective is to invoke the library rather than a conventional web page, several adjustments to the default configuration are necessary.

### Removing the `app` Folder

The `app` folder is redundant, as we'll be directly invoking the library instead of the `AppComponent`. Consequently, it can be safely removed.

### Adding `main.config.ts`

Create a `main.config.ts` file within the `src` directory and populate it with the following code:

```tsx
import { ApplicationConfig } from '@angular/core';

export const appConfig: ApplicationConfig = {
    providers: [
      // add code here
    ],
};
```

Add any necessary providers within this file.

## Installing `@angular/elements`

The `createCustomElement()` function from `@angular/elements` is essential for transforming the library into a Web Component.  Install `@angular/elements` to gain access to this function.

```bash
npm i @angular/elements
```

### Modifying `main.ts`

Replace the entire contents of the existing `main.ts` file with the code below:

```tsx
import {createApplication} from '@angular/platform-browser';
import {appConfig} from './main.config';
import {createCustomElement} from '@angular/elements';
import { WebComponentLibComponent } from 'web-component-lib';
import { ApplicationRef } from '@angular/core';

(async () => {
  const app: ApplicationRef = await createApplication(appConfig);

  // Define Web Components
  const webComponentLibComponent = createCustomElement(WebComponentLibComponent, {injector: app.injector});
  customElements.define('web-component-lib', webComponentLibComponent);
})();
```

The first parameter of `customElements.define` dictates the name used to invoke the library externally.

### Modifying `index.html`

Remove the default `app-root` element and insert the name of the library to be invoked. Ensure this name matches the first parameter provided to `customElements.define`.

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <title>Web Component</title>
        <base href="/" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
    </head>
    <body>
        <web-component-lib></web-component-lib>
        <script src="./main.js" type="module"></script>
        <script src="./polyfills.js" type="module"></script>
    </body>
</html>
```

#### Input and Output Handling

If your library incorporates `@Input` and `@Output` properties, these can be utilized within the Web Component. Inputs are employed in a manner analogous to HTML attributes (note that camelCase input names should be represented in kebab-case). Outputs can be accessed by attaching event listeners.

Modify `index.html` as follows to illustrate input and output handling:

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <title>Web Component</title>
        <base href="/" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
    </head>
    <body>
        <web-component-lib set-value="'sample'"></web-component-lib>
        <script src="./main.js" type="module"></script>
        <script src="./polyfills.js" type="module"></script>
        <script>
          const el = document.querySelector('web-component-lib'); // or set id to the dom and find the id.
          el.addEventListener('onResult', (event) => {
            console.log({event});
          });
        </script>
    </body>

</html>
```

### Modifying `angular.json`

Remove `"outputHashing": "all"` or set it to `"none"`. If `"outputHashing"` is set to `"all"`, the file names (e.g., `main-[hash].js`) will include a hash that changes with each build, which needs to be prevented.

## Build Process

Build both the library and the application separately.  Defining scripts in `package.json` can streamline this process.

```json
    "build:lib": "ng build web-component-lib --configuration production",
    "build:app": "ng build web-component-app --configuration production",
```

## Execution

To verify the proper functionality of the built application, locate the app within the `dist` folder and serve it using a local HTTP server (e.g., `http-server`). Simply opening the file directly will not work.

If `http-server` is unavailable, you can use a browser extension such as [Simple WebServer](https://simplewebserver.org/download.html).

## Optimization

Examine the `index.html` file within the app folder in the `dist` directory. Notice that it references `main.js` and `polyfills.js` as scripts.  If you have multiple libraries, this can lead to naming conflicts and inconvenience. Let's improve this.

### Adding a Script File

Create a `scripts` folder and add a file named `postbuild-bundler.js`.

```jsx
async function loadModules() {
    await import('./polyfills.js');
    await import('./main.js');
  }

  loadModules();
```

### Modifying `angular.json`

Update the `angular.json` file:

```json
// angular.json
"scripts": [
    {
        "input": "./scripts/postbuild-bundler.js",
        "bundleName": "bundle",
        "inject": false
    }
]
```

### Modifying `package.json`

Add another script to `package.json` to execute the script file concurrently with the build process:

```json
"build:all": "npm run build:lib && npm run build:app",
```

### Build

Execute the following command to build the application and simultaneously generate the script file:

```bash
npm run build:all
```

### Modifying `index.html`

Now, remove the script tags referencing `main.js` and `polyfills.js` from the `index.html` file within the app folder of the `dist` directory, and include only `bundle.js`.

```html
...
<script src="bundle.js" type="module"></script>
<!-- remove those codes
  <script src="./main.js" type="module"></script>
  <script src="./polyfills.js" type="module"></script>
-->
...
```

## Packaging

The library can be packaged separately, allowing for flexible configurations tailored to your environment.

## Import Methods

For information on consuming Web Components within Angular applications, refer to the [Web Components In Angular](https://coryrylan.com/blog/using-web-components-in-angular) article. Additional documentation may be provided separately as required.

## Considerations

To utilize the built Web Component, the `main.js`, `polyfills.js`, and `bundle.js` files must be included. Combining these into a single file would be more convenient, but a suitable solution has not yet been identified.

Referencing documentation suggesting the use of `fs-extra` and `concat` to merge `main.js` and `polyfills.js` into a single file resulted in errors due to duplicated variables, rendering it unusable.