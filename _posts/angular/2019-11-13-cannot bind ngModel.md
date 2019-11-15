---
title: "Cannot bind to 'ngModel' since it isn't a known property of 'input'"
date: 2019-10-17 12:23:00 +0900
comments: true
categories: angular
tags: [error]
---

### 원인
form 태그에 ngModel을 넣었을 때 Cannot bind to 'ngModel' since it isn't a known property of 'input' 에러가 발생합니다.


### 환경
- Angular 8.x 버전에서 발생하였습니다.


### 해결
form을 사용하기 위해서는 반드시 모듈에 FormsModule을 선언해주어야 하는데 이는 '@angular/forms'로부터 끌어와야 합니다.

- '@angular/forms'에서 FormsModule을 import
- NgModule의 imports 목록에 FormsModule 추가

```typescript
	import { FormsModule } from '@angular/forms';

	@NgModule({
		imports: [
			FormsModule
		]
	})
```