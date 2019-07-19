---
title: "Angular8 New Feature"
date: 2019-07-19 12:07:00 +0900
comments: true
categories: angular
tags: [svg, webworker, viewchild, formgroup]
---


Angular8에서는 기대하던 Ivy Rendering이 추가되고, 그 외에 많은 개선 및 추가 기능들이 포함되었습니다.
그 중에서 Angular를 제대로 활용하기 위해 반드시 알아야 할 새 기능과 변경된 기능을 추려서 정리해보겠습니다.
이해하지 못해 빠진 부분도 있으므로 댓글로 알려주시면 감사하겠습니다.


### SVG를 template으로

이제 component의 template을 설정할 때 html 외에도 svg 파일을 설정할 수 있습니다.

```ts
@Component({
  selector: 'app-root',
  templateUrl: './app.svg'
})
```


### Lazy Loading 의 변경

Routing 시 Lazy Loading 방식이 변경되었습니다.

#### 기존
```ts
loadChildren: './app.module#AppModule';
```

#### 변경
```ts
loadChildren: () => import('./app.module').then(m => m.AppModule)
```

자세한 사항은 [Angular 8 New Lazy Loading Routing](https://ksrae.github.io/angular/routing-lazy-loading/) 을 참고 하시기 바랍니다.




### WebWorker 생성

기존에는 angular 영역에서 벗어나 별도의 파일로 생성해야 했는데 이제 cli 환경에서 직접 webworker를 만들 수 있습니다.

```command
ng g webWorker [worker-name]
```


### FormGroup 기능 추가

#### markAllAsTouched

formGroup에 속한 모든 input 태그를 touch 상태로 만들어 주는 기능입니다.

```ts
formGroup.markAllAsTouched()
```

기존에는 formControl마다 markAsTouched 함수는 있었으나 모두 적용 함수도 추가되었습니다. 이로서 모두 적용 시에는 for문을 돌려가며 작성할 필요 없이 한줄의 코드로 해결할 수 있게 되었습니다.

#### clear

FormArray는 FormControl을 배열로 담아 관리하는 배열인데 기존에는 removeAt 함수로 배열에 담긴 FormControl의 전체/개별 삭제를 진행하였다면 이제 모두 제거 함수가 추가되어 한번에 삭제가 가능해졌습니다.

```ts
formArray.clear()
```


### ViewChild 변경

ViewChild 호출 시 static을 설정할 수 있게 되었습니다. 이로서 기존에 애매했던 호출 시점을 명확히 잡을 수 있게 되었습니다.
단, 호출되는 Directive나 Component에 ngIf 등의 동적 랜더링 조건이 들어간다면 static은 동작하지 않습니다.
이제 ViewChild는 ngAfterViewInit 이전에 호출됩니다.

```ts
@ViewChild(selector: directive type / querying name, {read: boolean, static: boolean})
```

selector에는 아래의 값이 들어갈 수 있습니다.
> template의 reference 변수값 (<my-component #cmp></my-component> 을 @ViewChild('cmp')으로 호출)

> child component가 가진 provider (@ViewChild(SomeService) someService: SomeService)

> string 토큰으로 정의된 provider ( @ViewChild('someToken') someTokenVal: any)

> TemplateRef (e.g. query <ng-template></ng-template> with @ViewChild(TemplateRef) template;)




이상 알아보았습니다. 성능의 개선과 테스트 코드의 수정 등 몇가지 더 변경된 점이 있으나 여기에서는 기능만 추려 정리하였기에 제외하였습니다.
끝.