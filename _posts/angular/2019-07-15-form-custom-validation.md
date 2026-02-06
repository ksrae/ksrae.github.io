---
title: "Form Custom Validation"
date: 2019-07-15 12:20:00 +0900
comments: true
categories: angular
tags: [form, validation, custom]
---

Let's explore how to implement custom validations beyond the default offerings. This demonstration will leverage a simple login form to illustrate applying regular expressions and creating new validators.

First, we define the template. Employing Angular Material facilitates the display of error messages. Therefore, the following template utilizes Angular Material components.

```html
<form [formGroup]="loginForm" (ngSubmit)="submit($event)">
  <mat-form-field>
    <input type="text" formControlName="id" placeholder="ID" tabindex="1" />
    <mat-error>Please enter your ID.</mat-error>
  </mat-form-field>

  <mat-form-field>
    <input type="password" formControlName="password" placeholder="Reset Login P/W" tabindex="4" />
    <mat-error>8-14 characters, including uppercase/lowercase letters, numbers, and special characters</mat-error>
  </mat-form-field>

  <mat-form-field>
    <input type="password" formControlName="passwordConfirm" placeholder="Enter P/W again." tabindex="5" />
    <mat-error>Please verify your P/W.</mat-error>
  </mat-form-field>

  <button type="submit">Confirm</button>
</form>
```

## Component Implementation

```tsx
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
  ) {}

  ngOnInit() {
    this.loginForm = this.fb.group({
      id: ['', [Validators.required]],
      password: ['', [Validators.required, Validators.pattern(this.validatorService.passwordValidator)]],
      passwordConfirm: ['', [Validators.required, this.validatorService.equalTo('password')]]
    });
  }
}
```

In the preceding code, a regular expression is applied to the `password` field:

```tsx
password: ['', [Validators.required, Validators.pattern(this.validatorService.passwordValidator)]],
```

Additionally, a custom validator, not leveraging built-in `Validators`, is applied to `passwordConfirm`:

```tsx
passwordConfirm: ['', [Validators.required, this.validatorService.equalTo('password')]]
```

Now, let's define the service.

## Service Definition

```tsx
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

`passwordValidator` incorporates a regular expression. Numerous excellent resources comprehensively explain regular expressions; therefore, an explanation is omitted here.

The `equalTo` function receives a `field_name` and returns a `ValidatorFn`. If the value of the control with the given `field_name` among all controls matches the current control's value, it returns `null` (indicating valid); otherwise, it returns `{ equalTo: false }`.

## Testing

Let's verify that the following conditions are correctly applied:

- All fields should be mandatory because `Validators.required` is applied to them. An error message should appear on each field if this condition is not met.
- The `password` field should display an error message if the regular expression condition is not satisfied.
- The `passwordConfirm` field should display an error message, triggered by the `equalTo` function, if its value does not match the `password` field's value.

## Results

If all tests function correctly, the validations are properly implemented.

Alternative methods exist for applying validations beyond the code presented. Validation can be checked within the `valueChange` event for real-time application, or it can be deferred to the submission event.

Choose the method that best suits your project's requirements.

End.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/%ED%8F%BC-validation-Customizing)
<br/>