---
title: JAMStack과 SSG (Definition of JAMStack and SSG)"
date: 2021-01-25 14:06:00 +0900
comments: true
categories: angular
tags: [ssg]
---


## 1. 개요
JAMStack은  기존 용어인 ‘LAMPStack (Linux, Apache, MySql, PHP)’ 혹은 ‘MEANStack (MongoDB, ExpressJS, AngularJS, NodeJS)’ 같은 단어와 비슷한 최근 웹 기술에서 구성해야할 요소의 모음입니다.<br/>
<br/>
JAMStack은 2018년 넷틀리파이에서 제안한 표현으로 서버의 요청을 최소화 하여 성능 및 보안 등에서의 높은 효과를 가져오기 위한 방법으로 제안되었으며, Javascript, APIs, Markup 의 앞글자를 딴 것입니다. 이는 기존 SPA의 방향과 많은 부분에서 일치하나 몇 가지 추가적인 기능을 포함합니다. <br/>
<br/>
정적 사이트 생성 (Static Site Generator)<br/>
CDN 의 활용<br/>

![cdn](https://t1.daumcdn.net/cfile/tistory/99C3E0435F3641130F)


(전통적인 웹 개발 구조와 JAMStack 구조의 차이)



CDN의 도입은 말 그대로 웹 서버를 대신할 목적이므로 크게 논의할 바 없으며, 사이트 환경에 따라 사용이 불가한 곳이 있으므로 여기에서는 논외로 하고, 

이 글에서는 정적 사이트 생성에 대해서 자세히 알아보겠습니다.



## 2. SSG (Static Site Generator)
### 2-1. 배경
SSG를 알아보기 전 배경을 이해하면 SSG의 도입 이유를 쉽게 이해할 수 있으므로 배경에 대해서 알아볼 필요가 있습니다.

웹 개발자라면 CSR과 SSR에 대해 들어보셨을 것이라 생각합니다. 이 둘의 특징은 다음과 같습니다.

SSR: 서버에서 html 페이지를 모두 구성하는 방식. 첫 페이지 로딩을 빠르게 사용자에게 전달할 수 있으며, SEO에 강합니다. 
단, 서버에서 html을 만들어야 하기 때문에 TTFB(Time to First Byte)가 느리며, 서버 자원을 사용하므로 서버에 부하를 줄 수 있으며, 요청이 잦아 트래픽이 높아질수록 성능이 떨어지며 공격에 취약합니다.
![SSR](https://unicorn-utterances.com/b3d14065d4f3d7e5aa6de108174946eb/ssr.svg)

CSR: SPA(Single Page Application) 방식이 도입되면서 랜더링 부분을 클라이언트에서 모두 구성하는 방식으로 TTFB가 빠르며, 서버 요청 횟수가 낮아 서버 부담을 줄이고 트래픽의 영향을 덜 받습니다.
단, SEO에 매우 취약하며, 전체적인 페이지 완료 시점은 SSR보다 느려지며, 클라이언트의 성능에 의존하므로 시점의 속도 차가 발생합니다.
![CSR](https://unicorn-utterances.com/6ef74dd32a6c239ddddab157667aa542/csr.svg)


### 2-2. 정의
SSG는 CSR의 SEO의 취약함을 보완하고, SSR의 서버 부하 문제를 해결하기 위해서 제안된 방식입니다. 

SSG는 빌드 타임에 정적 페이지를 pre-rendering하여 이를 클라이언트에게 전달하며, 이 후 CSR 방식으로 동작하도록 하는 방식입니다.

CDN에 쉽게 호스팅할 수 있으며, SEO 설정이 가능하다는 장점이 있으나, query data에 액세스할 수 없으므로 정적인 페이지인 경우 유리합니다. (이는 라이브러리에서 해결할 수 있으나 각각 다릅니다.)
![SSG](https://unicorn-utterances.com/241bc1a087bc9659d71d3655a36d1718/ssg.svg)






SSG는 쉽게 이해하면 SSR과 CSR의 중간쯤으로 보이나 엄밀히 따지면, 목적이 엄연히 다릅니다. SSG의 목적은 Headless에 있으며, 이는 Front-end, Back-end의 개념과는 다른, 생성된 하나의 페이지가 API 조차 필요 없이 어디서나 서비스 가능한 환경을 만드는 것에 그 목적이 있습니다. 따라서, 전형적인 서버-클라이언트 방식에서 벗어나고자 CDN과 같은 환경의 Serverless를 선호하며, 최소한의 요청으로 다양하고 많은 서비스를 할 수 있다는 점에서 비용면에서 클라우드 환경에서 매우 유리한 방식이라고 볼 수 있습니다.





## 3. 생태계 
가장 유명한 SSG는 jekyll이 있으며, 깃허브 블로그 사용자들이 주로 사용하고 있습니다.

React 쪽에서는 다양한 라이브러리가 있는데 그중 Gatsby와 next.js가 있으며, Vue.js에서는 next.js를 개조한 nuxt.js가 주로 쓰이고 있습니다.

Angular 진영에서는 이렇다 할 라이브러리가 없다가 2019년 말 개발된 Scully가 각광받고 있습니다. Universal이 SSR을 표방한다면, Scully는 next.js와 같이 SSG의 방식을 따르고 있어 Universal과 대조되고 있습니다.



## 4. Scully 동작 방식
Angular 코드 빌드 시 Scully의 머신 러닝을 통해 라우팅을 모두 검색하여 라우팅의 첫 페이지들을 모두 정적 html로 pre-rendering합니다. 

이 때, Scully는 크롬 브라우저의 Puppeteer를 사용하여 Angular 앱을 실행하여 모든 페이지를 연 뒤 IdleMonitorService를 통해 zone.js를 기반으로 스냅샷이 완료되었는지 여부를 결정합니다.

이를 통해 pre-rendering된 결과를 확인하고, 디버깅이 가능하도록 지원 합니다.

