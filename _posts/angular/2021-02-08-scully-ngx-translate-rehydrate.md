---
title: "Prevent rehydration using ngx-translate on scully"
date: 2021-02-08 14:37:00 +0900
comments: true
categories: angular
tags: [ngx-translate, rehydrate, scully]
---


## Task

This document outlines the process of applying `ngx-translate` to a Scully-based Angular application and addressing the rehydration issue that arises when refreshing the page after changing the language. Specifically, it details how to ensure that the statically generated HTML reflects the user's chosen language from the initial page load.

## Objective

1. Implement `ngx-translate` within a Scully application.
2. Identify the rehydration problem where the initial static HTML is generated in the default language, and then the translated content loads after a brief delay when refreshing the page.
3. Configure the application to serve the correct language-specific static HTML from the very first render.

## Implementation Plan

1. Utilize `LocalStorage` to persist the user's selected language.
2. Ensure that the initially served HTML incorporates the language setting stored in `LocalStorage`.

## Detailed Steps

### Implementing Internationalization with ngx-translate

Begin by installing `ngx-translate` and its HTTP loader. The HTTP loader is necessary for loading translation files.

```
npm i @ngx-translate/core @ngx-translate/http-loader
```

Next, configure `ngx-translate` in `app.config.ts`. Critically, set the `defaultLanguage` to the value retrieved from `localStorage` if available, defaulting to 'ko' (Korean) otherwise.

```tsx
// src/app/app.config.ts
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter } from '@angular/router';
import { HttpClient, provideHttpClient } from '@angular/common/http';
import { TranslateLoader, TranslateModule } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';
import { routes } from './app.routes';

export function createTranslateLoader(http: HttpClient) {
  return new TranslateHttpLoader(http, './assets/i18n/', '.json');
}

const defaultLanguage = (typeof window !== 'undefined' && window.localStorage.getItem('lang')) || 'ko';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(), 
    importProvidersFrom(
      TranslateModule.forRoot({
        defaultLanguage: defaultLanguage,
        loader: {
          provide: TranslateLoader,
          useFactory: createTranslateLoader,
          deps: [HttpClient],
        },
      })
    ),
  ],
};
```

Create `ko.json` and `en.json` files in the `assets/i18n` directory to hold the Korean and English translations, respectively. The following example contains a longer Korean translation for easier visual identification during testing.

```json
// ko.json
{
  "lang": "KoreanKoreanKoreanKoreanKoreanKorean"
}
// en.json
{
  "lang": "eng"
}
```

Add language selection buttons and a translated text element to the component's template.

```ts
// src/app/app.component.ts
import { Component, inject } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { TranslateModule, TranslateService } from '@ngx-translate/core';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, TranslateModule], 
  template: `
    <button (click)="changeLanguage('ko')">KO</button>
    <button (click)="changeLanguage('en')">EN</button>
    
    <p>{{ 'lang' | translate }}</p>

    <router-outlet></router-outlet>
  `,
})
export class AppComponent {
  private translateService = inject(TranslateService);

  constructor() {
    const lang = localStorage.getItem('lang') ?? 'ko';
    this.translateService.setDefaultLang(lang);
    this.translateService.use(lang);
  }

  changeLanguage(lang: string): void {
    localStorage.setItem('lang', lang);
    this.translateService.use(lang);
  }
}
```

At this point, running Scully and changing the language to English before refreshing will demonstrate the initial problem: the statically generated Korean HTML is displayed briefly before the English translation is loaded. This confirms the presence of the rehydration issue.

## Solution: Utilizing ngx-translate-router

To resolve the issue, install `@gilsdav/ngx-translate-router`.

```
npm i @gilsdav/ngx-translate-router
```

### 1. Adding LocalizeRouterModule to app.config.ts

Import and configure `LocalizeRouterModule` in `app.config..ts`.  It's crucial that `TranslateModule` and `RouterModule` are declared before `LocalizeRouterModule`.

```tsx
// src/app/app.config.ts
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter } from '@angular/router';
import { HttpClient, provideHttpClient } from '@angular/common/http';
import { TranslateLoader, TranslateModule, TranslateService } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';
import { LocalizeRouterModule, LocalizeRouterSettings, LocalizeParser } from '@gilsdav/ngx-translate-router';
import { routes } from './app.routes';

// ... (omitted) ...

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    importProvidersFrom(
      TranslateModule.forRoot({
        defaultLanguage: defaultLanguage,
        loader: {
          provide: TranslateLoader,
          useFactory: createTranslateLoader,
          deps: [HttpClient],
        },
      }),

      LocalizeRouterModule.forRoot(routes, {
        parser: {
          provide: LocalizeParser,
          useFactory: (translate: TranslateService, location: Location, settings: LocalizeRouterSettings) =>
            new ManualParserLoader(translate, location, settings, ['en', 'ko']), 
          deps: [TranslateService, Location, LocalizeRouterSettings],
        },
        initialNavigation: true,
      })
    ),
  ],
};
```

### 2. Setting skipRouteLocalization to app.routes.ts
```ts
// src/app/app.routes.ts
import { Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { LoginComponent } from './login/login.component';

export const routes: Routes = [
  // localized
  { path: 'home', component: HomeComponent },
  
  // skip localized
  { 
    path: 'login', 
    component: LoginComponent, 
    data: { skipRouteLocalization: true } 
  },
  
  // apply localized redirect only
  { 
    path: 'logout', 
    redirectTo: 'login', 
    data: { skipRouteLocalization: { localizeRedirectTo: true } } 
  },

  { path: '', redirectTo: 'home', pathMatch: 'full' }
];
```

Running Scully again, changing the language to English, and refreshing the page should now result in the correct English HTML being served immediately from the static file, resolving the rehydration problem.

## Conclusion

`@gilsdav/ngx-translate-router` operates by localizing routes. It generates routes for all supported languages and redirects the user to the appropriate language-specific route.

The library intercepts the requested route and redirects to the localized route (e.g., `/home` is transformed to either `/ko/home` or `/en/home` based on the current language selected by `ngx-translate`).

A prefix is employed to separate route conversion from standard application conversions. You can prevent segments from being translated and remove the prefix by using the `escapePrefix`.

The library automatically detects the locales chosen by `ngx-translate` and adjusts accordingly.

```tsx
this.translate.setDefaultLang();
this.translate.use();
```

## References

- [ngx-translate-router](https://github.com/gilsdav/ngx-translate-router)