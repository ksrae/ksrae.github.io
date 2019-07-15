---
title: "Form Group Custom Error 처리"
date: 2019-07-12 17:31:00 +0900
comments: true
categories: angular
tags: [form, formcontrol, formgroup, error, custom]
---


Angular의 FormControl 이나 FormGroup을 Material과 연동해서 사용하다보면 에러처리를 커스터마이징 해야할 때가 있습니다.
예를 들자면 동의 체크박스가 반드시 체크 되어야 하므로 체크가 되지 않으면 에러 메시지를 보여주어야 한다는 것 등입니다.

물론 별도로 검사하고 css 처리를 통해 메시지를 표시할 수 있겠으나 최대한 기본 설정을 유지한 채로 커스터마이징한 에러 처리를 하는 방법을 찾느라 여기저기 많은 사이트를 찾아보다가
이 방법이 가장 나을거 같아서 정리해보았습니다.




### 1. FormControl에서 에러처리

 FormControl 에러 발생 방법은 해당 control에 setError를 주고 mat-error에 해당 필드의 hasError를 체크하면 됩니다.


```html
{% raw %}
<mat-form-field>
      <input [formControl]="testControl" #testControl type="text" />
      <mat-error *ngIf="testControl.hasError('testError')"*>에러발생</mat-error>
</mat-form-field>
{% endraw %}
```

```ts
// 에러 강제 발생
this.testControl.setError({testError: true});
```


### 2. FormGroup에서 에러처리


```html
{% raw %}
<form [formGroup]="testForm">
      <mat-form-field  class="full-width">
            <input  matInput  type="text"  formControlName="checkControl"  />
            <mat-error>* 인증코드를 확인해 주세요</mat-error>
      </mat-form-field>
</form>
{% endraw %}
```

```ts
// component
...
ngOnInit() {
  this.testForm  =  this.fb.group({
    checkControl: ['', []]
  });
}

customError() {
  this.testForm.get('checkControl').markAsTouched();
  this.testForm.get('checkControl').setErrors({incorrect: true});
}
```


### 3. 유의점
> - 에러는 touch된 상황에서만 발생하므로 반드시 markAsToucked()를 우선 실행해야 합니다. 그렇지 않으면 setErrors 적용이 무시됩니다.
> - 에러를 취소하려면 setError(null) 또는 setError({incorrect: false}) 로 만들면 됩니다.
> - formGroup가 정상처리 되려면 모든 에러가 null 상태이어야 합니다.
> - 다른 예시 중에서 this.testForm.get('checkControl').updateValueAndValidity({onlySelf: true, emitEvent: false}); 도 있었으나 적용하면 동작하지 않았습니다. 혹시 되시면 댓글 부탁 드립니다.