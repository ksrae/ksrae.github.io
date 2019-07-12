---
title: "Form Group"
date: 2019-07-12 13:20:00 +0900
comments: true
categories: angular
tags: [form, formgroup]
---


Html에서 Form은 자주 사용되는데 Angular에서는 이 사용법이 은근 쉽지 않습니다.

몇 가지 설정과 세팅을 해주어야 비로소 유용하고 편리하게 사용할 수 있는데 이 과정이 매번 프로젝트 생성할 때마다 맞는지 꼼꼼히 확인하지 않으면 에러가 발생합니다.

앞으로도 자주 찾아볼거 같아 한 번 정리해두려고 합니다.


### 1. Module 등록

form을 사용할 component를 포함하고 있는 module이나 공통으로 사용하는 shared module에 ReactiveFormsModule을 import 합니다.



```ts

import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
      imports: [
            ReactiveFormsModule
      ]
})
```


그리고 일반 html 태그가 아닌 angular전용 태그를 사용할 것이므로 CUSTOM_ELEMENTS_SCHEMA도 해당 module에 추가해주어야 합니다.

```ts

import { NgModule, CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';

@NgModule({
  schemas: [
    CUSTOM_ELEMENTS_SCHEMA
  ]
})
```

### 2. template 작성

사용할 form을 작성합니다.

```html
{% raw %}
<form class="input-form" [formGroup]="searchForm" (ngSubmit)="submit($event)">
      <input type="text" formControlName="item" placeholder="검색어를 입력하세요" tabindex="1" />
      <button type="submit">검색</button><!--활설화 class="on" -->
</form>
{% endraw %}

```

위의 코드를 정리하면 다음과 같습니다.
- {% raw %} [formGroup] {% endraw %}에는 component에서 사용할 변수명을 입력합니다.
- (ngSubmit) 에는 submit을 수행할 함수명을 입력합니다.
- formControlName 에는 해당 input의 값을 처리할 변수명을 입력합니다.


### 3. component 작성

이제 component 작성만 하면 됩니다.

```ts
import { FormGroup, Validators, FormBuilder } from '@angular/forms';

@Component({
 ...
})
export class AppComponent {
    searchForm: FormGroup;

    constructor(
          private fb: FormBuilder
    ) {
          this.searchForm = this.fb.group({
                item: ['', [Validators.required]]
          });

          this.searchForm.valueChanges.subscribe(observer => {
                console.log(this.searchForm.valid);
          });          
    }

      submit(e) {
            const { item } = this.searchForm.controls; 
            console.log(item.value);
      }
}
```


먼저 FormGroup, Validators, FormBuilder를 import 합니다.
Validators는 사용하지 않거나 template에서 validation을 처리한다면 호출하지 않아도 됩니다.

            import { FormGroup, Validators, FormBuilder } from '@angular/forms';

template에서 작성한 FormGroup명을 변수로 선언합니다.

            searchForm: FormGroup;

FormBuilder를 DI로 가져옵니다. 

            constructor(
                  private fb: FormBuilder
            )


onInit이나 constructor에 formgroup을 정의합니다.

          this.searchForm = this.fb.group({
                item: ['', [Validators.required]]
          })

이 때, 사용되는 모든 form의 formControlName을 입력합니다. 
'' 부분은 초기값으로 초기값이 있으면 입력하고, 없으면 비워둡니다.
Validation은 배열 형태로 구성하되 없으면 빈 배열로 두어도 됩니다.


기본 제공하는 Validators의 종류는 아래와 같습니다.

```ts
class Validators {
  static min(min: number): ValidatorFn
  static max(max: number): ValidatorFn
  static required(control: AbstractControl): ValidationErrors | null
  static requiredTrue(control: AbstractControl): ValidationErrors | null
  static email(control: AbstractControl): ValidationErrors | null
  static minLength(minLength: number): ValidatorFn
  static maxLength(maxLength: number): ValidatorFn
  static pattern(pattern: string | RegExp): ValidatorFn
  static nullValidator(control: AbstractControl): ValidationErrors | null
  static compose(validators: ValidatorFn[]): ValidatorFn | null
  static composeAsync(validators: AsyncValidatorFn[]): AsyncValidatorFn | null
}
```

참고: https://angular.io/api/forms/Validators


valueChanges 이 부분은 form 값을 subscribe 하여 값의 변화를 실시간으로 체크하는 부분입니다.

          this.searchForm.valueChanges.subscribe(observer => {
                console.log(this.searchForm.valid);
          });   

끝으로 submit 함수에는 form 입력 이후 validation 체크나 데이터를 가공하는 등의 처리를 합니다.

          submit(e) {
                const { item } = this.searchForm.controls; 
                console.log(item.value);
          }

끝.