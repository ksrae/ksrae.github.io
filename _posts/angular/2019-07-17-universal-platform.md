---
title: "Angular Universal Platform Check"
date: 2019-07-17 14:07:00 +0900
comments: true
categories: angular
tags: [universal, platform]
---



Angular가 실행 중인 플랫폼이 브라우저인지 서버인지 확인하는 코드 입니다.<br><br>

확인하는 이유는 Angular가 서버 플랫폼에서 실행 중인 경우 브라우저에서만 사용 가능한 명령인 window, location, document 등을 사용할 수 없기 때문입니다.<br><br>

이 글은 angular universal이 적용된 프로젝트가 있다는 가정하에 작성하므로, angular universal 적용 방법은 [Angular Universal](https://ksrae.github.io/angular/angular-universal)을 참고하시기 바랍니다.


## 원리

프로젝트가 실행되면 PLATFORM_ID가 생성 되는데 이를 @angular/common에서 지원하는 isPlatformBrowser와 isPlatformServer 함수에 파라미터로 던져 그 결과로 판단할 수 있습니다.


## component

```ts
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

## template

```html
{% raw %}
<p>isPlatformBrowser: <span>{{isBrowser}}</span></p>
<p>isPlatformServer: <span>{{isServer}}</span></p>
{% endraw %}
```

이제 실행해보면 어떤 플랫폼에서 실행되었는지 알 수 있습니다.


끝.