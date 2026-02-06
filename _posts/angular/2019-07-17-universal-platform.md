---
title: "Angular Universal Platform Check"
date: 2019-07-17 14:07:00 +0900
comments: true
categories: angular
tags: [universal, platform]
---

Checking whether the platform Angular is running on is a browser or a server.

The reason for checking is that if Angular is running on a server platform, commands that are only available in the browser, such as window, location, and document, cannot be used.

This post assumes that an Angular Universal project is already set up, so please refer to [Angular Universal](https://ksrae.github.io/angular/angular-universal) for how to apply Angular Universal.

## Principle

When the project is executed, a `PLATFORM_ID` is generated, which can be passed as a parameter to the `isPlatformBrowser` and `isPlatformServer` functions supported by `@angular/common` to determine the platform.

## Component

```tsx
import { OnInit, Component, Inject, PLATFORM_ID } from '@angular/core';
import { isPlatformBrowser, isPlatformServer } from '@angular/common';

@Component({
  selector: 'app-root',
  templateUrl: './app.html'
})
export class AppComponent implements OnInit {
	isBrowser: boolean = false;
	isServer: boolean = false;

  constructor(
	@Inject(PLATFORM_ID) private platformId: object
  ) {

  }

  ngOnInit() {
	if(isPlatformBrowser(this.platformId)) this.isBrowser = true;
	if(isPlatformServer(this.platformId)) this.isServer = true;
  }
}
```

## Template

```html
<p>isPlatformBrowser: <span>{{isBrowser}}</span></p>
<p>isPlatformServer: <span>{{isServer}}</span></p>
```

Now, running the application will show on which platform it is being executed.

End.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular-Universal%EC%97%90%EC%84%9C-%ED%94%8C%EB%9E%AB%ED%8F%BC%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EC%97%AC%EB%B6%80-%EC%B2%B4%ED%81%AC)
<br/>