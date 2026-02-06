---
title: "Difference between Server-side Routing and Client-side Routing"
date: 2019-07-04 12:22:00 +0900
comments: true
categories: angular
tags: [routing]
---

This post compares routing implementations between server-side and client-side approaches.

## Key Differences

### Server-side Routing
Server-side routing handles navigation requests on the server. When a user accesses a page, the request reaches the server, which then renders the appropriate page and returns it to the client. This is the traditional way web applications worked before Single Page Applications (SPAs) became popular.

### Client-side Routing
Client-side routing handles navigation within the client application itself. When calling a specific page, it loads sequentially from the root to the target page. The routing is managed on the client side after the web application has been initially loaded.

## Similarities

### Server-side
When using View Layouts in frameworks like ASP.NET MVC, you can overlay views in a way similar to client-side routing. However, there's no clear module separation in this case, and layouts simply serve as views without dedicated controllers.

### Client-side
Each layout component has its own components and provides independent functionality. This enables better modularity and composition on the client side.

## Important Considerations

### Server-side
- Forced navigation must use server routing mechanisms
- Server controls page rendering
- Each navigation triggers a full page reload
- Better for initial page load SEO

### Client-side
- Must traverse through parent components when calling child routes
- Requires careful planning of component hierarchy
- Provides smoother user experience with no full page reloads
- May require additional SEO considerations

## Routing Implementation Examples

### Server-side Routing Example
Here's a simple server-side routing example using Node.js and Express.js:

```javascript
const express = require('express');
const app = express();

// Home page route handler
app.get('/', (req, res) => {
  res.send('Home Page');
});

// About page route handler
app.get('/about', (req, res) => {
  res.send('About Page');
});

// User profile page with parameters
app.get('/user/:id', (req, res) => {
  res.send(`User Profile Page - ID: ${req.params.id}`);
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Client-side Routing Examples

#### Traditional Angular Module-based Routing

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home.component';
import { AboutComponent } from './about.component';
import { UserProfileComponent } from './user-profile.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'user/:id', component: UserProfileComponent },
  { path: '**', redirectTo: '' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

```html
<!-- app.component.html -->
<nav>
  <ul>
    <li><a routerLink="/">Home</a></li>
    <li><a routerLink="/about">About</a></li>
    <li><a [routerLink]="['/user', '1']">User Profile</a></li>
  </ul>
</nav>

<router-outlet></router-outlet>
```

#### Angular Standalone Component Routing (Angular 14+)

```typescript
// app.routes.ts
import { Routes } from '@angular/router';
import { HomeComponent } from './home.component';
import { AboutComponent } from './about.component';
import { UserProfileComponent } from './user-profile.component';

export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'user/:id', component: UserProfileComponent },
  { path: '**', redirectTo: '' }
];
```

```typescript
// app.component.ts
import { Component } from '@angular/core';
import { RouterModule } from '@angular/router';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, RouterModule],
  template: `
    <nav>
      <ul>
        <li><a routerLink="/">Home</a></li>
        <li><a routerLink="/about">About</a></li>
        <li><a [routerLink]="['/user', '1']">User Profile</a></li>
      </ul>
    </nav>
    <router-outlet></router-outlet>
  `
})
export class AppComponent { }
```

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes)
  ]
}).catch(err => console.error(err));
```

## Key Takeaways

1. **Server-side Routing**:
   - Better for traditional multi-page applications
   - Each route request triggers a server round-trip
   - Simpler to implement for basic websites
   - Better initial SEO performance

2. **Client-side Routing**:
   - Ideal for single-page applications (SPAs)
   - Smoother user experience with no page refreshes
   - More complex to implement but offers better interactivity
   - Requires additional consideration for SEO

Choose your routing strategy based on your application's needs, considering factors such as user experience, SEO requirements, and application complexity.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/%EC%84%9C%EB%B2%84%EC%99%80-%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8%EC%97%90%EC%84%9C-%EA%B5%AC%EC%84%B1%ED%95%98%EB%8A%94-%EB%9D%BC%EC%9A%B0%ED%8C%85%EC%9D%98-%EB%B9%84%EA%B5%90-kwx2z2sz)
<br/>