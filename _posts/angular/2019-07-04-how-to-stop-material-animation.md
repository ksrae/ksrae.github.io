---
title: "특정 Material 애니메이션 중지 (How to Stop Angular Material Animation)"
date: 2019-07-04 15:45:00 +0900
comments: true
categories: angular
tags: [material, animation]
---


Angular Material에 기본적으로 적용된 모든 애니메이션이나 <br>특정 Material의 애니메이션만 중지하고 싶을 때, 컨트롤 하는 방법입니다.<br>



## 해결

여러 사이트를 많이 참고하였는데 해결 방법은 태그에 [@.disabled]="true" 를 넣어주는 것입니다.

```html
{% raw %}
<mat-tab-group [@.disabled]="true">
</mat-tab-group>
{% endraw %}
```

* 단, mat-tab-group 안의 mat-tab-group 에 적용할 경우 풀려 버리는 버그가 있으므로 <br>반드시 최상위 mat-tab-group에 적용해야 합니다.