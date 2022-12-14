---
title: "Form Custom Validation"
date: 2019-07-15 12:20:00 +0900
comments: true
categories: angular
tags: [form, validation, custom]
---


기본으로 제공되는 validation외에 cusom validation을 사용하는 방법을 알아보겠습니다.<br>
여기에서는 정규식 적용하기와 새로운 validation 적용하는 방법을 간단히 로그인 form을 활용해 알아보겠습니다.<br><br>



먼저 template을 작성합니다.<br>
material을 적용하면 에러 메시지를 쉽게 표현할 수 있으므로 여기에서는 angular material을 적용한 template을 작성하겠습니다.

```html
{% raw %}
<form [formGroup]="loginForm" (ngSubmit)="submit($event)">
      <mat-form-field>
            <input type="text" formControlName="id" placeholder="ID" tabindex="1" />
            <mat-error>ID를 입력해주세요.</mat-error>
      </mat-form-field>
      <mat-form-field>
            <input type="password" formControlName="password" placeholder="로그인 p/w 재설정" tabindex="4" />
            <mat-error>영문 대소문자, 숫자, 특수문자 포함 8 ~ 14자리</mat-error>
      </mat-form-field>
      <mat-form-field>
            <input type="password" formControlName="passwordConfirm" placeholder="P/W를 한번 더 입력하세요." tabindex="5" />
            <mat-error>입력하신 P/W를 한번 더 확인해주세요. </mat-error>
      </mat-form-field>
      <button type="submit">확인</button>
</form>
{% endraw %}
```

## component

```ts
import { Component, OnInit, ChangeDetectionStrategy, ChangeDetectorRef, NgZone } from '@angular/core';
import { FormGroup, Validators, FormBuilder } from '@angular/forms';
import { ValidatorService } from '../../services/validator.service';

  @Component({
      selector: 'app',
      templateUrl: './app.html',
      styleUrls: ['./app.css'],
      changeDetection: ChangeDetectionStrategy.OnPush
  })

export class AppComponent implements OnInit {
  loginForm: FormGroup;

  constructor(
    private fb: FormBuilder,
    private validatorService: ValidatorService,
    private cd: ChangeDetectorRef,
    private zone: NgZone
  ) {

  }

  ngOnInit() {
    this.loginForm = this.fb.group({
      id: ['', [Validators.required]],
      password: ['', [Validators.required, Validators.pattern(this.validatorService.passwordValidator)]],
      passwordConfirm: ['', [Validators.required, this.validatorService.equalTo('password')]]
    });
```

위의 코드를 보면 password에 정규표현식이 적용되어 있습니다.

      password: ['', [Validators.required, Validators.pattern(this.validatorService.passwordValidator)]],


또한 passwordConfirm에는 Validators를 활용하지 않는 새로 작성한 validation이 적용되어 있습니다.

      passwordConfirm: ['', [Validators.required, this.validatorService.equalTo('password')]]



이제 service를 작성하겠습니다.

## service

```ts
import { Injectable } from '@angular/core';
import { ValidatorFn, AbstractControl } from '@angular/forms';

@Injectable({
    providedIn: 'root'
})
export class ValidatorService {
    public passwordValidator: RegExp;

    constructor() {        
        this.passwordValidator = /^(?=.*[a-zA-Z])(?=.*\d)(?=.*[#$^+=!*()@%&]).[^/<>:\\]{6,12}$/;
    }

    public equalTo(field_name: string): ValidatorFn {
        return (control: AbstractControl): { [key: string]: any } => {
            let isValid = control.root.value[field_name] == control.value;
            if (!isValid) {
                return { 'equalTo': { isValid } };
            }
            return null;
        };
    }

}

```

passwordValidator는 정규 표현식이 적용되어 있습니다. 정규 표현식에 대해서는 설명이 잘 되어있는 좋은 사이트가 많으므로 여기에서는 설명을 생략하겠습니다.<br><br>

equalTo 함수를 보면 field_name을 받고, ValdatorFn 형식을 리턴합니다.<br>
모든 control 중 field_name을 가진 control의 값과 현재 control의 값이 일치하면 null (정상)을 일치하지 않으면 `{ equalTo: false }` 값을 리턴합니다.



## 테스트

아래의 조건이 올바로 적용되는지 테스트 해보겠습니다.
- 모든 필드에 Validators.required가 적용되어 있으므로 반드시 모든 필드를 채워야 합니다. 만일 그렇지 않으면 각 필드에 에러메시지가 표시되어야 합니다.
- password에 정규 표현식 조건이 걸려 있으므로 정규 표현식의 조건과 다른 경우 password 필드에 에러 메시지가 표시되어야 합니다.
- password와 passwordConfirm이 같은 값이 아니라면 equalTo 함수에 의해 passwordConfirm 필드에 에러 메시지가 표시되어야 합니다.



## 결과

모든 테스트가 올바르게 동작한다면 제대로 적용된 것입니다.<br><br>

위의 코드와 다른 방법으로도 validation을 적용할 수 있습니다.<br>
실시간 적용하기 위해 valueChange에서 validation을 체크할 수도 있고,<br>
실시간 적용을 포기하고 submit 시 할 수도 있습니다. <br><br>

프로젝트에 알맞은 방법을 찾아 적용하시기 바랍니다.<br><br>

끝.