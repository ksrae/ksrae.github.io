---
title: "No provider for ControlContainer"
date: 2019-10-17 12:23:00 +0900
comments: true
categories: angular
tags: [error]
---

### 원인
form 태그에 formGroup을 설정하였는데 'No provider for ControlContainer' 에러가 발생하였습니다.


### 환경
- Angular 8.x 버전에서 발생하였습니다.
- Angular Material을 적용하였습니다.


### 해결
formGroup 외에 angular에서 사용하는 form 명령을 사용하려면 module에서 ReactiveFormsModule 을 import 해야 합니다.
ReactiveFormsModule는 '@angular/forms'에 있습니다.

```typescript
	import { ReactiveFormsModule } from '@angular/forms';

	@NgModule({
		imports: [
			ReactiveFormsModule
		]
	})
```