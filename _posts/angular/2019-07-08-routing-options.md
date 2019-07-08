---
title: "Routing Options"
date: 2019-07-08 11:30:00 +0900
comments: true
categories: angular
tags: [routing]
---



Angular Routing Options


[Angular Docs](https://angular.io/api/router/NavigationExtras)

```ts
interface NavigationExtras {
  relativeTo?: ActivatedRoute | null
  queryParams?: Params | null
  fragment?: string
  preserveQueryParams?: boolean
  queryParamsHandling?: QueryParamsHandling | null
  preserveFragment?: boolean
  skipLocationChange?: boolean
  replaceUrl?: boolean
  state?: {...}
}
```

1) relativeTo?: ActivatedRoute | null	
링크가 어디에 기초하는지 정의

```ts
// this.route.root, .parent 등 다양한 기준이 있으므로 절대/상대 설정에 따라 정의.
this.router.navigate(['../list'], { relativeTo: this.route }); 
```


2) queryParams?: Params | null	
URL에 파라미터 붙이는 방법

```ts
// Navigate to /results?page=1
this.router.navigate(['/results'], { queryParams: { page: 1 } });
```


3) fragment?: string	
 frament를 붙이는 방법

```ts
// Navigate to /results#top
this.router.navigate(['/results'], { fragment: 'top' });
```   
        

4) preserveQueryParams?: boolean	
 다른 주소로 이동할 때 현재 가진 파라미터를 전달.
 7버전부터 폐기되므로 대신 queryParamsHandling 를 사용.

```ts
// Preserve query params from /results?page=1 to /view?page=1
this.router.navigate(['/view'], { preserveQueryParams: true });
```
        
        
5) queryParamsHandling?: QueryParamsHandling | null	
쿼리 파라미터 관리 확장버전.

```ts
// from /results?page=1 to /view?page=1&page=2
this.router.navigate(['/view'], { queryParams: { page: 2 },  queryParamsHandling: 'merge' });
 ```

- merge: 현재 URL에 포함된 파라미터를 함께 전달
- preserve: 현재 파라미터만 전달
- default: 기본형. 일반적인 방법으로 파라미터를 사용

 
6) preserveFragment?: boolean	
현재 fragment를 전달할지 여부

```ts
// Preserve fragment from /results#top to /view#top
this.router.navigate(['/view'], { preserveFragment: true });
```


7) skipLocationChange?: boolean	
 history에 포함하지 않고 이동. back할 때 이동하지 않음.

```ts
// Navigate silently to /view
this.router.navigate(['/view'], { skipLocationChange: true });
```   
        
        
8) replaceUrl?: boolean	
Navigates while replacing the current state in history.
url을 현재 주소로 교체할 것인지 여부. 

```ts
// Navigate to /view
this.router.navigate(['/view'], { replaceUrl: true });
 state?: {
    [k: string]: any;
}	
```


