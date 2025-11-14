---
title: "How to clear route made with outlet name attribute"
date: 2023-12-26 20:03:00 +0900
comments: true
categories: angular
tags: [routing]
---

Managing routes for dynamically injected components using Route Outlet Names in Angular applications is a powerful method for efficient UI construction. However, removing the current route from the outlet requires a different approach than standard route removal. This article discusses three methods for removing a route created with the Outlet's Name attribute.

## Setting the Outlet's Name Attribute Value to Null

Setting the Outlet's Name attribute value to null is a simple and intuitive method. To remove the current route from the outlet, you can simply set the Outlet's Name attribute value to null, without creating a new route in the browser's address bar.

```tsx
// app-popup.component.ts

@Component({
  selector: 'app-popup',
  standalone: true,
  imports: [RouterModule],
  template: `
    <dialog open>
      <h2>Popup</h2>
      <p>This is a popup content.</p>
      <a [routerLink]="'/', {outlets: {popupType: null}}]">Close</a>
    </dialog>
  `,
  styleUrls: ['./popup.component.css'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class PopupComponent {}
```

This approach leverages the Angular Router's ability to interpret a `null` value in the `outlets` configuration as a signal to clear the specified outlet. This allows for a clean and direct way to remove the component associated with that route.

## Removing with Route Navigation

Using `Router.navigate` to remove the current route from the outlet can be more complex, but it allows for more granular control. This method allows you to remove the current route at a specific time.

```tsx
// app-popup.component.ts

@Component({
  selector: 'app-popup',
  standalone: true,
  imports: [RouterModule],
  template: `
    <dialog open>
      <h2>Popup</h2>
      <p>This is a popup content.</p>
      <button (click)="close()">Close</button>
    </dialog>
  `,
  styleUrls: ['./popup.component.css'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class PopupComponent {
  router = inject(Router);
  route = inject(ActivatedRoute);

  close() {
    this.router.navigate(['/', { outlets: { popupType: null } }], {
      relativeTo: this.route.parent,
    });
  }
}
```

By injecting the `Router` and `ActivatedRoute` services, the component can programmatically trigger a navigation event that clears the `popupType` outlet. The `relativeTo: this.route.parent` option ensures that the navigation is performed relative to the parent route, maintaining the application's navigation context.

## Removing with Location

Using the `Location.back()` method to remove the current route operates similarly to the browser's back button behavior. This allows you to maintain consistency with the browser's basic navigation behavior.

```tsx
// app-popup.component.ts

@Component({
  selector: 'app-popup',
  standalone: true,
  imports: [RouterModule],
  template: `
    <dialog open>
      <h2>Popup</h2>
      <p>This is a popup content.</p>
      <button (click)="back()">Back</button>
    </dialog>
  `,
  styleUrls: ['./popup.component.css'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class PopupComponent {
  location = inject(Location); // @angular/common

  back() {
    this.location.back();
  }
}
```

The `Location` service from `@angular/common` provides an abstraction over the browser's history. Calling `location.back()` effectively navigates the user back to the previous route in the history stack, removing the current component from the outlet.

## Conclusion

Managing dynamic components using Route Outlet Names in Angular applications offers rich usability, but effectively removing the current route from the outlet requires different considerations than standard route removal. Choose and apply the appropriate method for your situation to achieve stable and efficient UI management.

## Complete Sample

First, create a component that calls the route.

```tsx
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterModule],
  templateUrl: `
  <a [routerLink]="['/', {outlets: {popupType: ['popup']}}]" >Open Popup</a>
  <router-outlet name="popupType"></router-outlet>
  `,
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'model-router-name-control';

}
```

Next, create a popup to expose via this route.

```tsx
@Component({
  selector: 'app-popup',
  standalone: true,
  imports: [
    RouterModule
  ],
  template: `
  <dialog open>
    <h2>Popup</h2>
    <p>This is a popup content.</p>

    <a [routerLink]="'/', {outlets: {popupType: null}}]">Close</a>
    <button (click)="back($event)">Back</button>
    <button (click)="close($event)">Close</button>
  </dialog>
  `,
  styleUrl: './popup.component.css',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class PopupComponent {
  loc = inject(Location); // @angular/common
  route = inject(Router);
  router = inject(ActivatedRoute);

  back(e: any) {
    this.loc.back();
  }
  close(e: any) {    
    this.route.navigate(['/', { outlets: { popupType: null } }], {
      relativeTo: this.router.parent
    });
  }
}
```