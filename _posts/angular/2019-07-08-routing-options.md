---
title: "Routing Options"
date: 2019-07-08 11:30:00 +0900
comments: true
categories: angular
tags: [routing]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Routing-Options)
<br/>

A comprehensive list of options for applying routing in Angular components.

[Angular Docs](https://angular.io/api/router/NavigationExtras)

```tsx
interface NavigationExtras {
  relativeTo?: ActivatedRoute | null
  queryParams?: Params | null
  fragment?: string
  preserveQueryParams?: boolean
  queryParamsHandling?: QueryParamsHandling | null
  preserveFragment?: boolean
  skipLocationChange?: boolean
  replaceUrl?: boolean
  state?: { [k: string]: any }
}
```

## relativeTo?: ActivatedRoute | null

> Defines the basis for the link. In essence, it specifies which `ActivatedRoute` the navigation is relative to. This enables relative navigation within the application.
> 

```tsx
// Several baselines are available such as this.route.root and .parent; define based on absolute/relative configurations.
this.router.navigate(['../list'], { relativeTo: this.route });
```

## queryParams?: Params | null

> A method for appending parameters to the URL, providing a way to pass data through the route.
> 

```tsx
// Navigate to /results?page=1
this.router.navigate(['/results'], { queryParams: { page: 1 } });
```

## fragment?: string

> How to append a fragment (also known as an anchor) to the URL. This is commonly used for in-page navigation.
> 

```tsx
// Navigate to /results#top
this.router.navigate(['/results'], { fragment: 'top' });
```

## preserveQueryParams?: boolean

> While navigating to a different address, this allows the preservation of the current query parameters. Note that this has been deprecated since version 7; `queryParamsHandling` should be used instead.
> 

```tsx
// Preserve query params from /results?page=1 to /view?page=1
this.router.navigate(['/view'], { preserveQueryParams: true });
```

## queryParamsHandling?: QueryParamsHandling | null

> An extended version for managing query parameters, offering greater control over how they are handled during navigation.
> 

```tsx
// from /results?page=1 to /view?page=1&page=2
this.router.navigate(['/view'], { queryParams: { page: 2 }, queryParamsHandling: 'merge' });
```

Options include:

- `merge`: Passes parameters included in the current URL along with the new parameters.
- `preserve`: Only passes the current parameters, discarding any new ones provided in the navigation.
- `default`: The standard behavior. Uses parameters in the conventional manner.

## preserveFragment?: boolean

> Determines whether the current fragment should be passed along during navigation.
> 

```tsx
// Preserve fragment from /results#top to /view#top
this.router.navigate(['/view'], { preserveFragment: true });
```

## skipLocationChange?: boolean

> Navigates without adding the route to the history. This means when the user clicks 'back', they won't navigate to this route.
> 

```tsx
// Navigate silently to /view
this.router.navigate(['/view'], { skipLocationChange: true });
```

## replaceUrl?: boolean

> Determines whether to replace the current URL with the new URL.
> 

```tsx
// Navigate to /view
this.router.navigate(['/view'], { replaceUrl: true });

state?: {
  [k: string]: any;
}
```