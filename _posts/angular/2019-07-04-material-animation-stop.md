---
title: "Angular Material Animation 멈추기"
date: 2019-07-04 15:45:00 +0900
comments: true
categories: angular
tags: [material, animation]
---

    태그에 [@.disabled]="true" 를 넣는다.

```
<mat-tab-group [@.disabled]="true">
</mat-tab-group>
```

* 단, mat-tab-group 안의 mat-tab-group 에 적용할 경우 풀려 버리는 버그가 있으므로 반드시 최상위 mat-tab-group에 적용해야 한다.