---
title: "Web Worker in Angular"
date: 2023-11-24 16:22:00 +0900
comments: true
categories: angular
tags: [webworker]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Angular%EC%97%90%EC%84%9C-Worker-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0)
<br/>

# Leveraging Workers in Angular Applications

Workers provide a mechanism to execute JavaScript code in background threads, thereby preventing blocking of the main thread and enhancing performance in Angular applications. This article explores the creation and utilization of workers within an Angular context.

## 1. Generating a Worker

For convenience, the Angular CLI provides a command to automatically generate worker files. Particularly, when utilizing Nx, it's highly recommended to use `ng generate` or `ng g`, as this automatically creates the `tsconfig.webworker.ts` file.

```
ng generate web-worker [path]
# example: ng generate web-worker core/workers/calculation
```

Note, however, that the auto-generated code directly accesses the worker from a component, differing from the instance-based approach described below. To align with the demonstrated code, some modification of the generated code is necessary.

Refer to "Manually Configuring Workers in an Nx Environment" at the end of this document for detailed instructions.

## 2. Crafting a Worker File

Begin by creating a dedicated worker file within your Angular project, typically using the `.worker.ts` extension. Here's a basic example:

```tsx
// calculation.worker.ts
addEventListener('message', ({ data }) => {
  console.log('Worker: message received->', data);
  
  // complex calculation in background
  const result = data * data;

  // return result to main thread
  postMessage(result);
});
```

## 3. Developing a Worker Service

Next, create an Angular service to manage the worker.

Typically, Angular services utilize `@Injectable` for dependency injection. However, to prevent worker sharing issues when the service is reused across components, it's advisable to instantiate a separate `Worker` instance for each component instead of injecting the service.

```tsx
// worker.service.ts
import { signal, WritableSignal } from '@angular/core';

export class WorkerService {
  private worker: Worker;

  // state management
  public readonly result: WritableSignal<number | undefined> = signal(undefined);
  public readonly isLoading: WritableSignal<boolean> = signal(false);
  public readonly error: WritableSignal<any | undefined> = signal(undefined);

  constructor(workerPath: string) {
    this.worker = new Worker(new URL(workerPath, import.meta.url), { type: 'module' });

    this.worker.onmessage = ({ data }) => {
      this.result.set(data);
      this.isLoading.set(false);
    };

    this.worker.onerror = (err) => {
      this.error.set(err);
      this.isLoading.set(false);
      console.error('Worker error:', err);
    };
  }

  runWorker(input: number): void {
    // initialize
    this.result.set(undefined);
    this.error.set(undefined);
    this.isLoading.set(true);
    
    // send start message to the Worker
    this.worker.postMessage(input);
  }

  // terminate worker when component is removed
  terminate(): void {
    this.worker.terminate();
    console.log('Worker is terminated.');
  }
}
```

## 4. Integrating the Worker in a Component

Finally, utilize the service created above within an Angular component to interact with the web worker.

```tsx
// app.component.ts
import { Component, OnDestroy } from '@angular/core';
import { WorkerService } from './worker.service';

@Component({
  selector: 'app-root',
  standalone: true,
  template: `
    <h2>Angular Worker with Signals</h2>
    <input #inputField type="number" value="10" />
    <button (click)="runWorker(inputField.valueAsNumber)">Run Worker</button>

    @if (worker.isLoading()) {
      <p>processing...</p>
    }

    @if (worker.result(); as result) {
      <p class="result">Result: {{ result }}</p>
    }

    @if (worker.error(); as error) {
      <p class="error">Error: {{ error.message }}</p>
    }
  `,
  styles: [`
    .result { color: green; font-weight: bold; }
    .error { color: red; }
  `]
})
export class AppComponent implements OnDestroy {
  readonly worker = new WorkerService('./calculation.worker.ts');
  
  constructor() {}

  runWorker(value: number) {
    if (isNaN(value)) {
      alert('inter any valid number.');
      return;
    }
    this.worker.runWorker(value);
  }

  ngOnDestroy(): void {
    this.worker.terminate();
  }
}
```

## Conclusion

You can now offload tasks to background threads using workers and transmit the results back to the main thread, leading to improved application responsiveness and overall performance.


## Optional: Manually Configuring Workers in an Nx Environment

### Manually Creating a `tsconfig` for Workers

Create a `tsconfig.worker.json` file in the same directory as your `tsconfig.json` file and insert the following code:

```json
/* To learn more about this file see: https://angular.io/config/tsconfig. */
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./out-tsc/worker",
    "lib": [
      "es2018",
      "webworker"
    ],
    "types": []
  },
  "include": [
    "src/**/*.worker.ts"
  ]
}
```

### Registering `tsconfig.webworker.json` in `angular.json`

You must register the `tsconfig.webworker.json` file created above in the `angular.json` file. When generated automatically, it is typically added to two locations:

- `architect > build > options`
- `test > options`

The value to add is:

```
"webWorkerTsConfig": "tsconfig.worker.json"
```

The complete `angular.json` code after automatic generation is as follows:

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "webworker": {
      "projectType": "application",
      "schematics": {
        "@schematics/angular:component": {
          "style": "scss"
        }
      },
      "root": "",
      "sourceRoot": "src",
      "prefix": "app",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "outputPath": "dist/webworker",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": [
              "zone.js"
            ],
            "tsConfig": "tsconfig.app.json",
            "inlineStyleLanguage": "scss",
            "assets": [
              "src/favicon.ico",
              "src/assets"
            ],
            "styles": [
              "src/styles.scss"
            ],
            "scripts": [],
            "webWorkerTsConfig": "tsconfig.worker.json"
          },
          "configurations": {
            "production": {
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "500kb",
                  "maximumError": "1mb"
                },
                {
                  "type": "anyComponentStyle",
                  "maximumWarning": "2kb",
                  "maximumError": "4kb"
                }
              ],
              "outputHashing": "all"
            },
            "development": {
              "buildOptimizer": false,
              "optimization": false,
              "vendorChunk": true,
              "extractLicenses": false,
              "sourceMap": true,
              "namedChunks": true
            }
          },
          "defaultConfiguration": "production"
        },
        "serve": {
          "builder": "@angular-devkit/build-angular:dev-server",
          "configurations": {
            "production": {
              "bro
```

```json
wserTarget": "webworker:build:production"
            },
            "development": {
              "browserTarget": "webworker:build:development"
            }
          },
          "defaultConfiguration": "development"
        },
        "extract-i18n": {
          "builder": "@angular-devkit/build-angular:extract-i18n",
          "options": {
            "browserTarget": "webworker:build"
          }
        },
        "test": {
          "builder": "@angular-devkit/build-angular:karma",
          "options": {
            "polyfills": [
              "zone.js",
              "zone.js/testing"
            ],
            "tsConfig": "tsconfig.spec.json",
            "inlineStyleLanguage": "scss",
            "assets": [
              "src/favicon.ico",
              "src/assets"
            ],
            "styles": [
              "src/styles.scss"
            ],
            "scripts": [],
            "webWorkerTsConfig": "tsconfig.worker.json"
          }
        }
      }
    }
  }
}
```