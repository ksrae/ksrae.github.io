---
title: "How to solve svg unsafe"
date: 2022-11-23 14:17:00 +0900
comments: true
categories: angular
tags: [svg, domaanitizer]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Angular%EC%97%90-svg-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0)
<br/>

To upload image files, the `<input type="file">` element must be used. After converting the result to a Base64 string, it can be displayed in an `<img>` tag.

## Adding a Standard Image

```html
<input type="file" (change)="addImage($event)">
<img [src]="imageSrc">
```

```tsx
addImage(file: FileList) {
   const reader = new FileReader();
   reader.readAsDataURL(file);
   reader.onload = () => {
      this.imageSrc = reader.result;  
   }
}
```

However, if you try to add an SVG file, the following error occurs, and a broken image is displayed:

```
 unsafe:data:image/svg+xml;base64,~~~
```

## Why Does an "Unsafe" Issue Occur When Attaching an SVG File?

Because SVG files can contain JavaScript, scripts can be injected externally. Therefore, browsers determine that this is a very insecure "unsafe" state and do not display the SVG image on the screen.

## Solution

You can resolve this issue by ensuring that the developer guarantees the safety of the SVG. Angular provides a built-in class called `DomSanitizer` to support this.

`DomSanitizer` provides various functions. To resolve the "unsafe" issue with SVGs, you must use `bypassSecurityTrustUrl()`.

```tsx
domSanitizer = inject(DomSanitizer);

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