---
title: "Difference between Server-side Routing and Client-side Routing"
date: 2019-07-04 12:22:00 +0900
comments: true
categories: angular
tags: [routing]
---


## 차이점
- Server-side: Server에서 Routing하기 때문에 해당 페이지에 직접 접근 하는 방식.
- Client-side: Client 내부에서 Routing 하기 때문에 해당 페이지를 호출 시 root 부터 해당 페이지까지 차례로 로딩.

## 유사점
- Server-side: Asp.net MVC의 View의 Layout을 활용하면 Client Routing처럼 View를 겹겹이 사용할 수 있으나 이는 module간 의 구분이 없고, Layout은 단순 View의 역할을 담당하며 해당 Controller가 없음.
- Client-side: 각각의 Layout도 Component를 가지며 독립적 기능이 있음.



## 주의사항
- Server-side: 원하는 페이지로 강제로 이동 시키고자 하는경우 반드시 routing을 사용해야 함.
- Client-side: 하위 페이지를 호출할 경우 해당 component의 부모 component를 무조건 거쳐야 하므로 설계 시 유의.