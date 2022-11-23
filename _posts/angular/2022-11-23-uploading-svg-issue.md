---
title: "How to solve svg unsafe"
date: 2022-11-17 14:17:00 +0900
comments: true
categories: angular
tags: [svg, domaanitizer]
---

이미지 파일을 업로드하기 위해서는 `<input type="file">`을 사용해야하며, 이 결과를 base64 string으로 변환하면 `<img>`에 표시할 수 있습니다.



## 일반 image의 추가

```
<input type="file" (change)="addImage($event)">
<img [src]="imageSrc">
```


```
addImage(file: FileList) {
   const reader = new FileReader();
   reader.readAsDataURL(file);
   reader.onload = () => {
      this.imageSrc = reader.result;  
   }
}
```



그런데 만일 svg 파일을 추가하면 다음과 같은 에러가 발생하며, 깨진 이미지를 볼 수 있습니다.

```
 unsafe:data:image/svg+xml;base64,~~~
```



## 왜 svg는 unsafe인가?

svg는 javascript를 포함할 수 있기 때문에 보안상 이슈가 있기 때문에 최신 브라우저는 이를 비정상적인 상황으로 보고 바로 표시하지 않습니다.



## 해결방법

unsafe를 개발자가 보장하면 이를 해결할 수 있습니다. angular에서는 이를 지원하기 위해 DomSanitizer 라고 하는 내장 클래스를 제공합니다.

DomSanitizer는 다양한 함수를 제공하는데 svg의 unsafe를 해결하기 위해서는 `bypassSecurityTrustUrl()` 를 활용해야 합니다.


```
constructor(
   private domSanitizer: DomSanitizer
)
addImage(file: FileList) {
   const reader = new FileReader();
   reader.readAsDataURL(file);
   reader.onload = () => {
      this.imageSrc = this.domSanitizer.bypassSecurityTrustUrl(reader.result as string);
   }
}
```




## Reference

[Unsafe SVG Issue](https://news.ycombinator.com/item?id=10626575)
