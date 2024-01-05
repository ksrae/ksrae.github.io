---
title: "특정 Material 애니메이션 중지 (How to Stop Angular Material Animation)"
date: 2019-07-04 15:45:00 +0900
comments: true
categories: angular
tags: [material, animation]
---


기본적으로 적용된 모든 애니메이션 또는 특정 Material 요소의 애니메이션을 중지하고자 할 때, 몇 가지 간단한 제어 방법을 사용할 수 있습니다.<br/>
<br/>

## 해결 방법

원하는 애니메이션을 중지하려는 대상 요소를 식별합니다.<br/>
<br/>
해당 요소에 애니메이션을 비활성화하기 위해 [@.disabled]="true" 속성을 추가합니다. <br/>
이로써 해당 요소의 애니메이션이 간단하게 중지됩니다.<br/>
<br/>
아래는 구체적인 사용 예시입니다.<br/>

```html
{% raw %}
<mat-tab-group [@.disabled]="true">
</mat-tab-group>
{% endraw %}
```

주의사항: 다만, 만약 애니메이션을 중지하고자 하는 요소가 다른 mat-tab-group 내부에 존재한다면, 이러한 설정이 오히려 원치 않는 버그를 초래할 수 있으니, 원활한 동작을 보장하기 위해 반드시 최상위 mat-tab-group에만 [@.disabled]="true" 속성을 적용해야 합니다. <br/>


