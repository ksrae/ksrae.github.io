---
title: "서버와 클라이언트에서 구성하는 라우팅의 비교 (Difference between Server-side Routing and Client-side Routing)"
date: 2019-07-04 12:22:00 +0900
comments: true
categories: angular
tags: [routing]
---

서버와 클라이언트에서 구성하는 라우팅의 비교 입니다.<br/>
<br/>
<br/>

## 차이점
Server-side (서버 측): 서버에서 라우팅을 처리하기 때문에 사용자가 해당 페이지에 직접 접근하는 방식입니다. 즉, 웹 페이지 요청이 서버에 도달하면 서버에서 해당 페이지를 렌더링하고 클라이언트에게 반환합니다.<br/>
<br/>
Client-side (클라이언트 측): 클라이언트 내부에서 라우팅을 처리하기 때문에 해당 페이지를 호출할 때 루트(root)에서 시작하여 해당 페이지까지 차례로 로딩합니다. 즉, 웹 애플리케이션이 로드된 후에 클라이언트 측에서 라우팅을 담당합니다.<br/>
<br/>
<br/>

## 유사점
Server-side (서버 측): ASP.NET MVC의 View Layout을 활용하면 클라이언트 측 라우팅과 유사한 방식으로 View를 겹쳐서 사용할 수 있습니다. 그러나 이 경우 모듈 간 구분이 없으며, 레이아웃은 단순히 뷰(View)의 역할을 하며 해당 컨트롤러가 없습니다.<br/>
<br/>
Client-side (클라이언트 측): 각각의 레이아웃(Layout)도 컴포넌트(Component)를 가지며 독립적인 기능을 제공합니다. 이것은 클라이언트 측에서 모듈화 및 구성성을 더 잘 활용할 수 있음을 의미합니다.<br/>
<br/>

## 주의사항
Server-side (서버 측): 원하는 페이지로 강제로 이동하려면 반드시 라우팅을 사용해야 합니다. 클라이언트 측과 달리 서버가 페이지 렌더링을 제어하므로 서버 라우팅을 적용해야 합니다.<br/>
<br/>
Client-side (클라이언트 측): 하위 페이지를 호출할 때 해당 컴포넌트의 부모 컴포넌트를 반드시 거쳐야 합니다. 따라서 설계 시에 이 점을 유의해야 합니다.<br/>
<br/>

## 라우팅(Server-side vs. Client-side) 비교

라우팅은 서버와 클라이언트 간의 웹 페이지 내비게이션을 다루는 중요한 주제입니다. 두 가지 주요 접근 방식을 비교해 보겠습니다.<br/>
<br/>

### 서버 측 라우팅(Server-side Routing)
서버 측 라우팅은 서버에서 라우팅을 처리하는 방식입니다. 예를 들어, 사용자가 웹 페이지에 접근하면 서버에서 해당 페이지를 렌더링하고 클라이언트에게 반환합니다. 다음은 Node.js와 Express.js를 사용한 간단한 서버 측 라우팅 예제 코드입니다.<br/>
<br/>

```javascript
const express = require('express');
const app = express();

// 루트 경로에 대한 핸들러
app.get('/', (req, res) => {
  res.send('홈 페이지');
});

// '/about' 경로에 대한 핸들러
app.get('/about', (req, res) => {
  res.send('소개 페이지');
});

app.listen(3000, () => {
  console.log('서버가 3000번 포트에서 실행 중입니다.');
});
```

### 클라이언트 측 라우팅(Client-side Routing)
클라이언트 측 라우팅은 웹 애플리케이션이 로드된 후 클라이언트 측에서 라우팅을 처리하는 방식입니다. 주로 JavaScript 프레임워크(예: Angular, React, Vue.js)를 사용하여 구현됩니다.<br/>

 
```typescript
// app-routing.module.ts 파일에서 라우트 설정을 추가합니다.
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home.component';
import { AboutComponent } from './about.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

```html
<!-- app.component.html -->
<nav>
  <ul>
    <li><a routerLink="/">홈</a></li>
    <li><a routerLink="/about">소개</a></li>
  </ul>
</nav>

<router-outlet></router-outlet>
```
