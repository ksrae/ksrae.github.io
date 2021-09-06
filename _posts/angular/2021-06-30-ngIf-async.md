---
title: "ngIf에서 여러 Observable 설정하기 (More than one Observable in a *ngIf statement)"
date: 2021-06-30 14:16:00 +0900
comments: true
categories: angular
tags: [ngIf, Observable]
---


## 계획
1. ngIf에서 하나의 Observable 정의합니다.
2. ngIf에서 둘 이상의 Observable 정의합니다.



## 하나의 Observable 적용
async를 적용하려면 변수명 뒤에 async pipe를 추가하고 as로 사용할 변수명을 정의해줍니다.

```html
{% raw %}
<div *ngIf="user$ | async as user">
  <p>{{user.name}}</p>
</div>
{% endraw %}
```

## 여러개의 Observable 적용
여러 개의 Observable을 정의할 때, 하나의 Observable과 같이 작성할 경우 에러가 발생합니다.

```html
{% raw %}
<!-- 에러 발생 -->
<div *ngIf="(user$ | async as user) || (item$ | async as item)"></div>
{% endraw %}
```


## 해결 - 여러개의 Observable 적용
여러개의 Observable 적용하기 위해서는 Object 형태로 변형하여 사용해야 합니다.<br>
아래의 코드는 정상 동작 합니다.

```html
{% raw %}
<div *ngIf="{user: user$ | async, item: item$ | async} as userInfo">
	<p>{{userInfo.user.name}}</p>
</div>
{% endraw %}
```



## 참고 사이트
- [ngx-translate-router](https://github.com/gilsdav/ngx-translate-router)

