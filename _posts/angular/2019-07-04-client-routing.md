---
title: "Difference between Server-side Routing and Client-side Routing"
date: 2019-07-04 12:22:00 +0900
comments: true
categories: angular
tags: [routing]
---

  ## Server-side vs. Client-side Routing

  Server-side routing involves handling requests on the server to generate complete pages that are returned to the client. This approach lends itself to robust SEO capabilities and often simpler state management, but it requires additional round-trips to the server for page updates.

  Client-side routing, on the other hand, relies on JavaScript frameworks to intercept URL changes and render views on the client. This allows for faster, more dynamic transitions once the application is loaded, though it may require extra steps to ensure proper SEO and initial page load performance.

  ### Example: Node.js and Express (Server-side)
  ```javascript
  const express = require('express');
  const app = express();

  app.get('/', (req, res) => {
    res.send('Home Page');
  });

  app.get('/about', (req, res) => {
    res.send('About Page');
  });

  app.listen(3000, () => {
    console.log('Server is running on port 3000');
  });
  ```

  ### Example: Angular (Client-side)
  ```typescript
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

  ### Additional Example with Standalone Components (Angular)
  ```typescript
  import { Component } from '@angular/core';
  import { RouterModule } from '@angular/router';

  @Component({
    standalone: true,
    imports: [RouterModule],
    selector: 'app-standalone-home',
    template: `
      <h1>Standalone Home</h1>
      <p>This standalone component illustrates a simple client-side route.</p>
    `
  })
  export class StandaloneHomeComponent {}
  ```

  Using standalone components can reduce complexity by removing the need for an NgModule. They also help streamline project structure, making it clearer how each route is tied to its own dedicated component.