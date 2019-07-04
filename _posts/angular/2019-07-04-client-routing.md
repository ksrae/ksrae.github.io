---
title: "Client Routing의 주의점"
date: 2019-07-04 12:22:00 +0900
comments: true
categories: angular
tags: [routing]
---


## 차이점
- 기존 Routing: Servier에서 Routing하기 때문에 해당 페이지에 직접 접근 하는 방식이다.
- Client Routing: Client 내부에서 Routing 하기 때문에 해당 페이지를 호출 시 root 부터 해당 페이지까지 차례로 로딩한다.

## 유사점
- Client Routing: 각각의 Layout도 Component를 가지며 독립적 기능이 있다.
- 기존 Routing: Asp.net MVC의 View의 Layout을 활용하면 Client Routing처럼 View를 겹겹이 사용할 수 있다. 그러나 이는 module간 의 구분이 없고, Layout은 단순 View의 역할을 담당하며 해당 Controller가 없다.


## 주의사항
- 기존 Routing: 원하는 페이지로 강제로 이동 시키고자 하는경우 반드시 routing을 사용하여야 한다.
- Client Routing: 하위 페이지를 호출할 경우 해당 component의 부모 component를 무조건 거쳐야 하므로 설계 시 유의 하여야 한다.