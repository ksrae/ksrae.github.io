---
title: "Apply Multiple Languages with ngx-translate"
comments: true
categories: angular
tags: [language, translate]
date: 2019-07-03 12:20:00 +0900
---

Apply multiple languages instantly using ngx-translate without server requests or reloading.<br>


# Implementing Internationalization with ngx-translate in Angular

This guide covers implementing internationalization in Angular applications using ngx-translate, supporting both NgModule and Standalone approaches. We'll explore advanced configuration options and best practices for enterprise-level applications.

## Installation

Install the required dependencies:

```bash
npm install @ngx-translate/core @ngx-translate/http-loader --save
```

## Implementation Approaches

### 1. NgModule-based Implementation

#### Core Module Configuration

```typescript
// app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpClientModule, HttpClient } from '@angular/common/http';
import { TranslateModule, TranslateLoader } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';

export function HttpLoaderFactory(httpClient: HttpClient) {
  return new TranslateHttpLoader(httpClient, 'assets/i18n/', '.json');
}

@NgModule({
  imports: [
    BrowserModule,
    HttpClientModule,
    TranslateModule.forRoot({
      loader: {
        provide: TranslateLoader,
        useFactory: HttpLoaderFactory,
        deps: [HttpClient]
      },
      defaultLanguage: 'en-US',
      useDefaultLang: true
    })
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### 2. Standalone Implementation

#### Bootstrap Configuration

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { HttpClient, provideHttpClient } from '@angular/common/http';
import { TranslateLoader, TranslateModule } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';
import { importProvidersFrom } from '@angular/core';

export function HttpLoaderFactory(httpClient: HttpClient) {
  return new TranslateHttpLoader(httpClient, 'assets/i18n/', '.json');
}

bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(),
    importProvidersFrom(
      TranslateModule.forRoot({
        loader: {
          provide: TranslateLoader,
          useFactory: HttpLoaderFactory,
          deps: [HttpClient]
        },
        defaultLanguage: 'en-US',
        useDefaultLang: true
      })
    )
  ]
}).catch(err => console.error(err));
```

#### Standalone Component Implementation

```typescript
// app.component.ts
import { Component, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { TranslateModule } from '@ngx-translate/core';
import { LanguageService } from './services/language.service';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    CommonModule,
    TranslateModule
  ],
  providers: [LanguageService],
  template: `
    <div [innerHTML]="'common.welcome' | translate"></div>
  `
})
export class AppComponent {
  // for standalone
  languageService = inject(LanguageService);

  // for module
  constructor(private languageService: LanguageService) {
    this.languageService.initialize();
  }
}
```

## Type-Safe Language Configuration

### Language Interface

```typescript
// interfaces/language.interface.ts
export enum LanguageCode {
  'en-US' = 'English',
  'ko-KR' = '한국어',
  'zh-CN' = '简体中文',
  'ja-JP' = '日本語'
}

export interface TranslationKeys {
  common: {
    welcome: string;
    login: string;
    logout: string;
  };
  errors: {
    required: string;
    invalid: string;
  };
}
```

### Language Service Implementation

```typescript
// services/language.service.ts
import { Injectable, inject } from '@angular/core';
import { TranslateService } from '@ngx-translate/core';
import { LanguageCode } from '../interfaces/language.interface';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class LanguageService {
  private currentLang = new BehaviorSubject<string>(null);
  currentLang$ = this.currentLang.asObservable();
  
  // for standalone
  translateService = inject(TranslateService);

  // for module
  constructor(private translateService: TranslateService) {}

  initialize(): void {
    const languages = Object.keys(LanguageCode);
    this.translateService.addLangs(languages);
    
    const browserLang = this.translateService.getBrowserLang();
    const defaultLang = languages.find(lang => 
      browserLang.toLowerCase() === lang.toLowerCase()
    ) || 'en-US';
    
    this.translateService.setDefaultLang(defaultLang);
    this.setLanguage(defaultLang);
  }

  setLanguage(lang: keyof typeof LanguageCode): void {
    this.translateService.use(lang);
    this.currentLang.next(lang);
  }

  getCurrentLang(): Observable<string> {
    return this.currentLang$;
  }
}
```

## Translation Files Structure

```json
// assets/i18n/en-US.json
{
  "common": {
    "welcome": "Welcome to our application",
    "login": "Login",
    "logout": "Logout"
  },
  "errors": {
    "required": "This field is required",
    "invalid": "Invalid input value"
  }
}
```

## Advanced Usage Examples

### 1. Dynamic Translation with Parameters

```typescript
// template usage
@Component({
  template: `
    <div [innerHTML]="'notifications.welcome' | translate:{ username: currentUser.name }"></div>
  `
})

// translation file
{
  "notifications": {
    "welcome": "Welcome back, {{username}}!"
  }
}
```

### 2. Implementing Language Switcher

```typescript
@Component({
  selector: 'app-language-switcher',
  standalone: true,
  imports: [CommonModule, TranslateModule],
  template: `
    <select (change)="onLanguageChange($event)" [value]="currentLang$ | async">
      <option *ngFor="let lang of languageCodes" [value]="lang">
        {{ LanguageCode[lang] }}
      </option>
    </select>
  `
})
export class LanguageSwitcherComponent {
  protected readonly LanguageCode = LanguageCode;
  protected readonly languageCodes = Object.keys(LanguageCode);
  protected currentLang$ = this.languageService.getCurrentLang();

  // for stadnalone
  languageService = inject(LanguageService);

  // for module
  constructor(private languageService: LanguageService) {}

  onLanguageChange(event: Event): void {
    const lang = (event.target as HTMLSelectElement).value as keyof typeof LanguageCode;
    this.languageService.setLanguage(lang);
  }
}
```

## Best Practices and Considerations

1. **Performance Optimization**
   - Use lazy loading for translation files in large applications
   - Implement caching strategies for loaded translations
   - Consider using translation compilation for production builds

2. **Type Safety**
   - Define strong types for translation keys
   - Use TypeScript interfaces for translation structures
   - Implement validation for translation files during build process

3. **Maintenance**
   - Organize translations in a hierarchical structure
   - Implement translation key management system
   - Use translation management tools for large projects

4. **HTML Content Handling**
   - Always use [innerHTML] for translations containing HTML
   - Implement sanitization for user-provided translation content
   - Consider accessibility implications when using HTML in translations

## Common Pitfalls and Solutions

1. **Nested HTML Elements**
   ```typescript
   // Incorrect
   <div>{{ 'key.with.html' | translate }}<span>Additional content</span></div>
   
   // Correct
   <div [innerHTML]="'key.with.html' | translate"></div>
   <span>Additional content</span>
   ```

2. **Async Translation Loading**
   ```typescript
   // Handle translation loading state
   this.translateService.get('key').subscribe({
     next: (translation: string) => {
       // Handle successful translation
     },
     error: (err) => {
       console.error('Translation error:', err);
       // Provide fallback
     }
   });
   ```

## Integration with Angular Features

1. **Route Guards**
   ```typescript
   @Injectable({ providedIn: 'root' })
   export class TranslationLoadGuard implements CanActivate {
    // for standalone
    translateService = inject(TranslateService);

    // for module
    constructor(private translateService: TranslateService) {}

    canActivate(): Observable<boolean> {
       return this.translateService.get('common.welcome')
         .pipe(map(() => true));
     }
   }
   ```

2. **Error Handling**
   ```typescript
   @Injectable({ providedIn: 'root' })
   export class GlobalErrorHandler implements ErrorHandler {
    // for standalone
    translateService = inject(TranslateService);

    // for module
    constructor(private translateService: TranslateService) {}

     handleError(error: Error): void {
       this.translateService.get('errors.unexpected')
         .subscribe(message => {
           // Handle error with translated message
         });
     }
   }
   ```


## Reference
- [ngx-translate](https://github.com/ngx-translate/core)


## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular%EC%97%90%EC%84%9C-ngx-translate%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EB%8B%A4%EA%B5%AD%EC%96%B4-%EC%B2%98%EB%A6%AC-%EA%B5%AC%ED%98%84-%EA%B0%80%EC%9D%B4%EB%93%9C)
<br/>