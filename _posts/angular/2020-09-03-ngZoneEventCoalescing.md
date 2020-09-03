---
title: "ngZoneEventCoalescing으로 이벤트 버블링 해결(ngZoneEventCoalescing for Preventing Event Bubbling)"
date: 2020-09-03 17:04:00 +0900
comments: true
categories: angular
tags: [bubbling]
---


## 환경
Angular 9

  
## 원인
js에는 부모와 자식간 이벤트 버블링이 발생하는데 대부분의 경우 이벤트 버블링 때문에 불 필요한 이벤트를 처리하는 등의 문제를 겪습니다.<br/>
Angular에서도 역시 이벤트 버블링이 발생하는데 이는 랜더링 성능에 영향을 끼치게 됩니다.<br/>
<br/>
이를 해결하기 위해 changeDetection을 설정하는데 9버전 이하에서는 changeDetection을 설정하여도 ngAfterViewInit에서 타이밍 이슈로 인해 setTimeout에 함수를 담아야 실행 되는 경우가 있었습니다.<br/>

```tsx
ngAfterViewInit(){  
    setTimeout(()=> {
        this.changeStatus()
    })  
}
```

Angular 9 버전부터 보다 근본적인 버블링 해결책을 구현하였으며, 또한 위의 setTimeout 코드는 changeDetection의 detectChange으로 이벤트를 문제없이 적용할 수 있습니다.<br/>

```tsx
ngAfterViewInit(){  
    this.changeStatus();
    this.cdr.detectChanges();  
}
```


## 해결

main.ts에서 Angular Module boostrap 설정 시 옵션을 추가합니다.

```tsx
   platformBrowserDynamic()
  .bootstrapModule(AppModule, { ngZoneEventCoalescing: true })
```



## 참고 사이트
[how-to-prevent-event-bubbling-in-angular-9-with-setinterval](https://stackoverflow.com/questions/60854223/how-to-prevent-event-bubbling-in-angular-9-with-setinterval)

[change-detection-strategy-in-angular](https://dev.to/gaurangdhorda/change-detection-strategy-in-angular-25mc)