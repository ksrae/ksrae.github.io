---
title: "No provider for ControlContainer"
date: 2019-10-17 12:23:00 +0900
comments: true
categories: angular
tags: [error]
---

### Cause

A 'No provider for ControlContainer' error occurred when setting formGroup in the form tag.

### Environment

- The issue occurred in Angular version 8.x.
- Angular Material was applied.

### Solution

To use form commands in Angular in addition to formGroup, you need to import ReactiveFormsModule in the module. ReactiveFormsModule is located in '@angular/forms'.

```tsx
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
    imports: [
        ReactiveFormsModule
    ]
})
```