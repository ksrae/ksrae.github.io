---
title: "Angular Universal Location"
date: 2019-07-17 14:37:00 +0900
comments: true
categories: angular
tags: [universal, location]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Angular-Universal%EC%97%90%EC%84%9C-window.location-%EB%AA%85%EB%A0%B9-%EC%82%AC%EC%9A%A9)
<br/>

How to Use `window.location` in Angular Universal Projects on the Server Platform

This document explains how to effectively use the `window.location` command when your Angular Universal project is running on the server platform.  We leverage the Location Service provided by `@angular/common`, eliminating the need to import external libraries.

## Component Implementation

```tsx
import { OnInit, Component } from '@angular/core';
import { Location } from '@angular/common';

@Component({
  selector: 'app-location',
  templateUrl: './location.html'
})
export class LocationComponent implements OnInit {

  constructor(
    private location: Location
  ) {

  }

  ngOnInit() {
    console.log(this.location.path());
  }

  goBack(e) {
    e.preventDefault();
    this.location.back();
  }
}
```

If you execute the code as-is, you might encounter an error similar to the one shown below:

`error TS2339: Property 'location' does not exist on type 'LocationComponent'.`

This error occurs because `Location` and `LocationStrategy` and/or `PathLocationStrategy` must be added to the `providers`.  While you *can* add them directly to the component, it is better practice to include them in the module, as this makes them available to multiple components.

## Module Configuration

```tsx
import { NgModule, CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';
import { Location, LocationStrategy, PathLocationStrategy} from '@angular/common';
import { LocationRouting } from './location.routing';
import { LocationComponent } from './location.component';

@NgModule({
  declarations: [
    LocationComponent
  ],
  imports: [
    LocationRouting
  ],
  exports: [

  ],
  providers: [
    Location,
    {provide: LocationStrategy, useClass: PathLocationStrategy}
  ],
  schemas: [
    CUSTOM_ELEMENTS_SCHEMA
  ]
})
export class LocationModule { }
```

Now, you should see that everything works correctly.

## Important Considerations

You might encounter the following error:

`error TS2740: Type 'Location' is missing the following properties from type 'Location': path, getState, isCurrentPathEqualTo, normalize, and 7 more.`

This issue can arise if you forget to include the `private` keyword when injecting `Location` in the component's constructor.

```tsx
  constructor(
    location: Location
  ) { }
```

The solution is to declare `location` as private:

```tsx
  constructor(
    private location: Location  // Must be declared as private.
  ) { }
```

## Reference Links

- [Angular Location API](https://angular.io/api/common/Location)
- [Using Angular's Location Service ← Alligator.io](https://alligator.io/angular/location-service/)