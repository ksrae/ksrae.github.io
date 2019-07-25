---
title: "Angular Universal Location"
date: 2019-07-17 14:37:00 +0900
comments: true
categories: angular
tags: [universal, location]
---



angular universal 프로젝트가 server platform에서 실행 중일 때 window.location 명령을 사용하는 방법 입니다.
@angular/common에서 지원하는 Location Service를 활용하는 방법이므로 별도의 라이브러리를 가져올 필요는 없습니다.


## component

```ts
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


만일 이대로 작성한 상태에서 실행하면 다음과 비슷한 에러가 발생합니다.

  error TS2339: Property 'location' does not exist on type 'LocationComponent'.


이는 Location과 LocationStrategy and/or PathLocationStrategy를 providers에 추가해야 하는데 component에 직접 추가해도 되나 module에 넣으면 여러 component에서 활용할 수 있으므로 module에 넣는 것이 좋습니다.

## module

```ts
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

이제 올바르게 동작하는 것을 확인할 수 있습니다.



## 유의점

다음과 같은 에러가 발생하는 경우도 있는데

  error TS2740: Type 'Location' is missing the following properties from type 'Location': path, getState, isCurrentPathEqualTo, normalize, and 7 more.


이는 component의 constructor에서 Location을 DI 하였을 때, private을 넣지 않아 발생하는 케이스입니다.

```ts
  constructor(
	location: Location
  ) { }
```

이는 location을 private으로 선언하면 해결 됩니다.

```ts
  constructor(
	private location: Location	 // private으로 선언해야 합니다.
  ) { }
```


참고: 
> [Angular](https://angular.io/api/common/Location)

> [Using Angular's Location Service ← Alligator.io](https://alligator.io/angular/location-service/)
