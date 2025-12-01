---
title: "Componentless Routing"
date: 2019-07-04 11:59:00 +0900
comments: true
categories: angular
tags: [routing]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EC%97%86%EC%9D%B4-%EB%9D%BC%EC%9A%B0%ED%8C%85-%EA%B5%AC%EC%84%B1)
<br/>

# Analysis
When setting up routing, components are typically configured as follows:
```js
{path: '', component: AppComponent, children: [
  { path: '', redirectTo: 'main', pathMatch: 'full' },
  { path: 'main', component: MainComponent },       
  ...
]}
```

When trying to access MainComponent using "/" or "/main", the request must first go through AppComponent before being redirected to MainComponent via the redirectTo configuration.<br/>
When using Bootstrap with this routing setup, a potential issue arises where AppComponent gets called twice due to the interaction between Bootstrap and the routing system. <br/>
This double initialization can lead to unnecessary overhead and potential complications.<br/>
Furthermore, if AppComponent serves merely as a parent component with only `<router-outlet></router-outlet>` and no additional functionality, we can utilize Componentless Routing to avoid creating this unnecessary component altogether.

# Solution
This issue can be resolved by implementing Componentless Routing.<br/>
Simply remove the component property from the root route configuration. <br/>
This allows direct access to the pages specified in the children's redirectTo:
```js
{path: '', children: [
  { path: '', redirectTo: 'main', pathMatch: 'full' },
  { path: 'main', component: MainComponent },
  ...
]}
```

Important considerations when using Componentless Routing:
1. Since there's no parent component, you must specify a default child route using redirectTo
2. When accessing MainComponent via "/" or "/main", it will be rendered directly in the `<router-outlet></router-outlet>` of the calling component<br/>
3. This results in a simpler and more efficient routing structure

## Additional benefits I'd like to add:

- Improved performance due to fewer component initializations
- Cleaner dependency injection hierarchy
- Reduced memory footprint
- Better compatibility with lazy loading
- Easier testing due to simpler component structure

Best practices when implementing Componentless Routing:

1. Always ensure proper route matching with pathMatch: 'full' for empty path routes
2. Consider using route guards if you need to implement security or loading logic that would have been in the parent component
3. If you need shared layout elements, consider using a layout component as one of the child routes instead of a parent component
4. Document the routing structure clearly for other developers, as componentless routes might be less immediately obvious in larger applications

This approach is particularly useful in:

- Single-page applications with simple routing needs
- Applications using micro-frontends where routing needs to be lightweight
- Projects where you want to minimize the component tree depth
- Situations where you need to avoid Bootstrap's double-initialization issues