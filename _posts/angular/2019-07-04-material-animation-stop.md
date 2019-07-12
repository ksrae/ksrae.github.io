---
title: "Angular Material Animation 멈추기"
date: 2019-07-04 15:45:00 +0900
comments: true
categories: angular
tags: [material, animation, template]
---


### 개요
Angular Material을 쓰다보니 애니메이션이 기본으로 적용되어 있는데 이를 멈추기 위해 css를 강제로 지정해도 제대로 되지 않는 경우가 있습니다.
또한 모든 Material이 아닌 특정 Material의 애니메이션만 중지하고 싶은데 이 경우에도 css 컨트롤이 쉽지 않습니다.


### 해결
여러 사이트를 많이 참고하였는데 해결 방법은 의외로 쉬웠습니다.

    태그에 [@.disabled]="true" 를 넣는다.

```html
{% raw %}
<mat-tab-group [@.disabled]="true">
</mat-tab-group>
{% endraw %}
```

* 단, mat-tab-group 안의 mat-tab-group 에 적용할 경우 풀려 버리는 버그가 있으므로 반드시 최상위 mat-tab-group에 적용해야 합니다.