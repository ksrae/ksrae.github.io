---
title: "trackBy Directive"
date: 2021-09-03 16:05:00 +0900
comments: true
categories: angular
tags: [ngfor, trackby]
---


ngFor에 연결된 배열의 값이 변경될 때 refresh가 발생하여 rendering이 일어나는데, 이를 방지하기 위해 trackBy를 사용합니다.<br/>
즉, trackBy를 사용하면 값이 변경되어도 rendering이 되지 않으므로 성능을 향상 시킬 수 있습니다.<br/>



## trackBy 사용 방법과 불편함
trackBy는 다음과 같이 사용합니다. 

```html
{% raw %}
<!-- app.component.html -->
<div *ngFor="let item of data; trackBy: trackByFn"></div>
{% endraw %}
```

```tsx
// app.component.ts
...
trackByFn = (item): number => item.id
```

trackBy의 단점은 위의 코드에서 보듯 ngFor문이 사용될 때마다 component에서 매번 유사한 함수를 만들어주어야 한다는 것입니다. <br/>
이 불편함 때문에 개발자들이 종종 trackBy를 고의로 누락하거나 실수로 작성하지 않습니다.<br/>

## trackBy 동작방식
먼저 trackBy가 어떻게 동작하는지 원리를 알아보아야 합니다. 아래의 두 코드는 동일한 결과를 갖습니다.

```html
{% raw %}
<div *ngFor="let item of data; trackBy: trackByFn"></div>

<ng-template ngFor let-item [ngForOf]="data" [ngForTrackBy]="trackByFn"></ng-template>
{% endraw %}
```

ng-template에서 보면 알 수 있듯이 trackBy의 원래 이름은 ngForTrackBy이며 ngFor directive의 전달인자로 볼 수 있습니다.


## trackBy를 Directive로 만들 때 문제점

 어떻게 ngForTrackBy를 directive화 할 수 있을까요? 일반적인 방법으로 생각해보면 다음과 같을 것 입니다.

```html
{% raw %}
<div *ngFor="let item of data" [trackByFn]="'id'"></div>
{% endraw %}
```

이를 ng-template에서 풀어보면 다음과 같이 됩니다.

```html
{% raw %}
<ng-template ngFor let-item [ngForOf]="data">
	<div [ngForTrackByFn]="'id'"></div>
</ng-template>
{% endraw %}
```

즉, ngFor 내부가 아닌 별도의 directive로 인식되므로 ngFor에 관여할 수 없게 됩니다. <br/>

그러면 어떻게 ngFor에 접근할 수 있을까요?<br/>


## Host를 활용한 ngForTrackBy Directive 만들기
@Host는 Angular의 내장된 기능으로 @Self와 유사하게 연관된 DI를 가져올 수 있습니다. <br/>

차이가 있다면, @Self는 component와 연관된 DI를 가져오는 반면 component의 template과 viewProviders에 연관된 DI 를 가져옵니다. <br/><br/>

즉, 우리는 @Host를 사용하여 ngFor를 가져오고 ngFor에 속한 ngForTrackBy 함수에 우리가 원하는 필드 값을 주입해야 합니다. 코드는 아래와 같습니다.<br/>

```tsx
import { NgForOf } from "@angular/common";
import { Directive, Host, Input } from "@angular/core";

@Directive({
  selector: "[ngForTrackByField]",
})
export class NgForTrackByFieldDirective<T> {
  
  @Input()
  public ngForTrackByField!: keyof T;

  constructor(@Host() private ngFor: NgForOf<T>) {
    this.ngFor.ngForTrackBy = (index: number, item: T) => {
      return this.ngForTrackByField ? item[this.ngForTrackByField] : item;
    };
  }
}
```

위의 코드에서 보면 ngForTrackByField는 ngFor를 호스트로 두는 directive이며, 호출되었을 때 ngFor의 ngForTrackBy함수에 원하는 값을 주입하고 실행시키는 역할을 합니다.<br/><br/>

이제 template에서는 trackByField를 통해 간단히 trackBy를 구현할 수 있습니다.<br/>

```html
{% raw %}
<div *ngFor="let item of data; trackByField: 'id'"></div>
{% endraw %}
```



## 참고 사이트
[Host](https://angular.io/api/core/Host) <br/><br/>
[Host Decorator in Angular](https://www.tektutorialshub.com/angular/host-decorator-in-angular/) <br/><br/>
[How you can use trackBy with a property name](https://medium.com/@ingobrk/here-is-how-you-can-use-trackby-with-a-property-name-ec3bbba8fa75) <br/><br/>


