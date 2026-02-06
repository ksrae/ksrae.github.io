---
title: "Execute Function on Init (App Initializer)"
date: 2019-07-12 15:01:00 +0900
comments: true
categories: angular
tags: [initializer, factory]
---

I aim to create a function that executes faster than a component at the point when an app is executed.

In project development, there's often a need to execute a function faster than a component to set or load values. For example, retrieving JSON values or setting whether to log.

In such cases, you can leverage `APP_INITIALIZER`.

First, create the function to operate. Here, I'll create a service to load a JSON file.

```tsx
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http'

@Injectable({
    providedIn: 'root'
})
export class ConfigService {

    constructor(private httpClient: HttpClient) {

    }

    public async get() {
        return new Promise((resolve, reject) => {
            this.httpClient.get('/assets/config.json',
            ).toPromise().then((result) => {
                resolve(result);
            }).catch(err => {
                resolve(null);
            });
        });
    }
}
```

The `get` function creates a service that reads a `config.json` file and resolves it as a Promise.

Now, modify the module that's executed first (typically `app.module`).

```tsx
import { NgModule, APP_INITIALIZER } from '@angular/core';
import { ConfigService } from '../services/config.service';

export function getConfig(configService: ConfigService) {
  return () => configService.get();
}

@NgModule({
      providers: [{
            provide: APP_INITIALIZER,
            useFactory: getConfig,
            deps: [ConfigService],
            multi: true
      }]
})
```

First, import `APP_INITIALIZER` from `@angular/core`.
Then, import the service to use. Here, I'll call `config.service`.

The service has been changed so that it's no longer called in the module, but since the service must be used in this case, the service was also imported.

```tsx
import { NgModule, APP_INITIALIZER } from '@angular/core';
import { ConfigService } from '../services/config.service';
```

Looking at `NgModule`, I've set `APP_INITIALIZER`, `ConfigService`, and a factory called `getConfig` in `providers`. The `multi` value is also set to `true`.

Here, you define the point at which it's provided, the service name to use, and which factory to utilize.

```tsx
{
    provide: APP_INITIALIZER,
    useFactory: getConfig,
    deps: [ConfigService],
    multi: true
}
```

Now, in the middle of the module code, you'll see that I've implemented a function called `getConfig`, which is the function defined in the factory.
The content is simply calling the `get` function of `configService`.

```tsx
export function getConfig(configService: ConfigService) {
    return () => configService.get();
}
```

Now everything is set. If you modify the code a little more to allow components to use the values, you can use the values set in the service at the start of the component.

End.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/%EC%95%B1-%EC%8B%A4%ED%96%89-%EC%8B%9C%EC%A0%90%EC%97%90-%ED%95%A8%EC%88%98-%ED%98%B8%EC%B6%9C-App-Initializer)
<br/>