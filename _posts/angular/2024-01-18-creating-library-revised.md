---
title: "Creating Private Library"
date: 2024-01-18 19:50:00 +0900
comments: true
categories: angular
tags: [library, package]
---

## Creating and Utilizing Local Angular Libraries: A Comprehensive Guide

In July 2019, I authored a document, "[Creating Library Document](https://ksrae.github.io//angular/creating-library/)," which, in retrospect, lacked clarity. I've rewritten it to enhance understanding.

This guide explores the process of developing and utilizing a custom-built Angular library as if it were an external, publicly available package.

With Angular 6 and later, the official support for `ng-packagr` simplifies the library creation process. Leverage this capability to encapsulate frequently used functionalities into reusable libraries and packages for enhanced code maintainability and efficiency.

## 1. Project Initialization

Begin by creating a standard Angular project using the Angular CLI:

```
ng new app
```

## 2. Adding a Library to Your Project

Employ the Angular CLI to generate a new library within your project:

```bash
ng generate library [library-name]
```

This command creates a `projects` directory in your Angular root folder, containing a subfolder named after your library (e.g., `library-name`). For the purpose of this guide, we'll assume the library name is `mylib`.

## 3. Library Development

Develop the desired functionality within your library. This involves creating components, services, and modules as needed.

## 4. Crafting the `public_api.ts` File

The `public_api.ts` file serves as the entry point for your library, exposing the components, modules (in module-based projects), and services you want to make accessible to consuming applications. Import and re-export all relevant components, modules, and services:

```tsx
export * from './lib/my-lib.service';
export * from './lib/my-lib.component';
export * from './lib/my-lib.module';
```

This file essentially defines the public interface of your library.

## 4-1. (Module-Based Projects Only) Adding Exports in the Module

In module-based Angular projects, declare all components within the `declarations` array of your module and export them in the `exports` array. Note that services are typically *not* included in the `exports` array. If your project uses standalone components this step can be skipped.

```tsx
import { NgModule } from '@angular/core';
import { MyLibComponent } from './my-lib.component';
import { FooComponent } from './foo/foo.component';

@NgModule({
  imports: [
  ],
  declarations: [
    MyLibComponent,
    FooComponent
  ],
  exports: [
    MyLibComponent,
    FooComponent
  ]
})
export class MyLibModule { }
```

## 5. Adding a Script to `package.json`

For streamlined packaging, add a custom script command to your project's `package.json` file:

```json
"scripts": {
  "packagr": "ng-packagr -p ng-package.json"
}
```

This script simplifies the process of building your library into a distributable package.

## 6. Building the Library

Execute the `packagr` script using npm:

```
npm run packagr
```

This command invokes `ng-packagr` to build your library.

## 7. Verifying the Output Directory

Upon successful build completion, a `dist/[library-name]` directory should be generated. In our example, this would be `dist/mylib`. This directory contains the packaged library artifacts.

## 8. Packaging the Library

Navigate to the `dist/[library-name]` directory and execute the following npm command:

```
npm pack
```

This command creates a `.tgz` archive of your library, ready for local installation.

## 9. Confirmation

Verify the creation of the `[library-name]-[version].tgz` file (e.g., `mylib-0.0.1.tgz`). This file represents the packaged version of your library.

With the library now packaged, let's integrate it into an Angular application.

## 10. Creating a New Project for Testing

Generate a new Angular project to consume the newly created library:

```
ng new newApp
```

## 11. Copying the `.tgz` File

Copy the `[library-name]-[version].tgz` file from the library's `dist` folder to a location within your new application's directory structure. The specific location is not critical.

In this guide, we'll create a `local-package` folder and place the `mylib-0.0.1.tgz` file within it.

## 12. Installing the `.tgz` File

Navigate to the directory containing the `.tgz` file (or remain at the project root if you placed it there) and install the library using npm:

```
npm i [library-name]-[version].tgz
```

For our example, this translates to:

```
npm i mylib-0.0.1.tgz
```

## 13. Verifying Installation in `package.json`

Open the `package.json` file of your application and confirm that the library is listed as a dependency:

```json
"dependencies": {
  [library-name]: "file:[folder]/[library-name]-[version].tgz"
}
```

In our example, you should see:

```json
"dependencies": {
  "mylib": "file:local-package/mylib-0.0.1.tgz"
}
```

If the entry is missing or the file path is incorrect, manually add or correct the dependency entry in `package.json`.

## 14. Utilizing the Library

Import and use the library's components within your application components. The selector used to reference the library's components is determined by the component's selector defined when you created the library.

In module-based Angular projects, you must import the library's module into your application module. Standalone components can be imported directly.

```tsx
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterOutlet } from '@angular/router';
import { MylibModule } from 'mylib'; // Adjust import path as needed

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, RouterOutlet, MylibModule],
  template: '<lib-mylib></lib-mylib>',
})
export class AppComponent {
  title = 'app-call-local-library';
}
```

## Conclusion

This guide has demonstrated a straightforward approach to creating local Angular libraries and utilizing them as private packages.

Since libraries can incorporate other external libraries, development and integration should be relatively seamless.

## Reflections

My previous attempt to condense a guide encompassing both public package publishing and local library creation resulted in an unstructured and confusing document. This revision reflects the difficulty in conveying information succinctly while maintaining clarity. I am committed to producing more intuitive and accessible documentation in the future.