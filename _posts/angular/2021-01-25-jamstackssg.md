---
title: "Definition of JAMStack and SSG"
date: 2021-01-25 14:06:00 +0900
comments: true
categories: javascript
tags: [ssg]
---

## 1. Overview

The JAMstack is a modern web development architecture, similar to terms like 'LAMP Stack (Linux, Apache, MySQL, PHP)' or 'MEAN Stack (MongoDB, ExpressJS, AngularJS, NodeJS)', that describes a collection of essential elements for building modern web applications.

Introduced by Netlify in 2018, the JAMstack aims to minimize server requests, enhancing performance and security. The acronym stands for Javascript, APIs, and Markup. It aligns with many aspects of Single Page Applications (SPAs) but includes additional functionalities.

Key components of the JAMstack include:

- Static Site Generation (SSG)
- Content Delivery Network (CDN) Utilization

[](https://t1.daumcdn.net/cfile/tistory/99C3E0435F3641130F)

cdn

*(Difference between traditional web development structure and JAMstack structure)*

While CDN implementation is primarily for replacing web servers and requires minimal discussion, its feasibility depends on the site environment and is excluded from further discussion here.

This article will delve into Static Site Generation (SSG).

## 2. SSG (Static Site Generator)

### 2-1. Background

Understanding the background of SSG is crucial to grasp its adoption rationale. A brief overview of the context is necessary.

Web developers are likely familiar with Client-Side Rendering (CSR) and Server-Side Rendering (SSR). Their key characteristics are as follows:

SSR: The server constructs complete HTML pages. This approach enables rapid initial page load and enhances SEO.
However, since the server generates the HTML, the Time to First Byte (TTFB) is slower. It also consumes server resources, potentially leading to server overload, performance degradation with frequent requests, and vulnerability to attacks.

[](https://unicorn-utterances.com/b3d14065d4f3d7e5aa6de108174946eb/ssr.svg)

SSR

CSR: With the introduction of Single Page Applications (SPAs), rendering is primarily handled on the client side. This results in faster TTFB, fewer server requests, reduced server load, and less sensitivity to traffic fluctuations.
However, CSR is SEO-unfriendly. The overall page completion time is slower than SSR, and performance depends on client-side capabilities, leading to varying speeds.

[](https://unicorn-utterances.com/6ef74dd32a6c239ddddab157667aa542/csr.svg)

CSR

### 2-2. Definition

SSG is a method proposed to mitigate CSR's SEO weaknesses and SSR's server load issues.

SSG pre-renders static pages at build time and delivers them to the client, subsequently operating in a CSR manner.

It offers easy hosting on CDNs and SEO configuration advantages. However, it's best suited for static pages because it cannot directly access query data (though libraries may offer solutions).

[](https://unicorn-utterances.com/241bc1a087bc9659d71d3655a36d1718/ssg.svg)

SSG

SSG can be loosely seen as a middle ground between SSR and CSR, but their purposes are distinctly different.
SSG's purpose lies in Headless architecture, aiming to create an environment where a generated page can be served anywhere without even requiring APIs, moving beyond the front-end and back-end paradigm.

Therefore, it favors serverless environments like CDNs, moving away from the typical server-client approach.
The ability to deliver diverse and numerous services with minimal requests makes it a cost-effective option in cloud environments.

## 3. Ecosystem

The most renowned SSG is Jekyll, widely used by GitHub blog users.

In the React ecosystem, Gatsby and Next.js are prominent libraries. For Vue.js, Nuxt.js, a modified version of Next.js, is commonly used.

The Angular community lacked a significant library until Scully, developed in late 2019, gained popularity. While Universal aims for SSR, Scully adopts an SSG approach, contrasting with Universal.

## 4. SSG in Angular

From Angular 9 onwards, Universal offers a prerender option that allows generating static HTML at build time, similar to SSG.

Therefore, projects already built with Universal version 9 or higher can be adapted simply by changing the build command, without requiring major restructuring.

### 4.1. Angular Universal Prerender

Verify support for prerendering by checking the `prerender` option in `angular.json` and the build options in `package.json`.

```tsx
// angular.json
"serve-ssr": {
    "builder": "@nguniversal/builders:ssr-dev-server",
    "options": {
    "browserTarget": "prerender-demo:build",
    "serverTarget": "prerender-demo:server"
    },
    "configurations": {
    "production": {
        "browserTarget": "prerender-demo:build:production",
        "serverTarget": "prerender-demo:server:production"
    }
    }
},
"prerender": {
    "builder": "@nguniversal/builders:prerender",
    "options": {
    "browserTarget": "prerender-demo:build:production",
    "serverTarget": "prerender-demo:server:production",
    "routes": [
        "/"
    ]
    },
    "configurations": {
    "production": {}
    }
```

Execute the build by verifying the `prerender` execution command in `package.json`.

```tsx
// package.json
"prerender": "ng run prerender-demo:prerender"
```

```bash
> npm run prerender
```

### 4.2. Scully

A disadvantage of Angular Universal Prerender is the continued need to check for server-side conditions, preventing immediate use of browser commands like `window`.

Scully operates solely on the client-side, allowing for easier coding when adopting SSG.

Scully's operation is as follows:

> - During Angular code build, Scully's machine learning scans all routes and pre-renders the first pages of these routes as static HTML.
> - At this point, Scully uses Chrome browser's Puppeteer to execute the Angular app, opening all pages and then using IdleMonitorService to determine whether the snapshot is complete, based on zone.js.
> - This enables verification of the pre-rendered results and supports debugging.


## Link
[한국어(Korean) Page](https://velog.io/@ksrae/JAMStack%EA%B3%BC-SSG)
<br/>