---
title: "Form Group"
date: 2019-07-12 13:20:00 +0900
comments: true
categories: angular
tags: [form, formgroup]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%ED%8F%BC-%EA%B7%B8%EB%A3%B9)
<br/>

## Implementing Angular Forms: A Comprehensive Guide

Working with forms in HTML is a common task. However, in Angular, the implementation might not be immediately straightforward. Several configurations and setups are required to use forms effectively and conveniently. Errors can occur if these steps aren't carefully verified each time a new project is created. This guide aims to provide a consolidated reference for implementing Angular forms.

## Module Registration

Import `ReactiveFormsModule` into the module containing the component that will use the form or into a shared module if the form is used across multiple components.

```tsx
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  imports: [
    ReactiveFormsModule
  ]
})
```

Additionally, since we'll be using Angular-specific form elements instead of standard HTML tags, add `CUSTOM_ELEMENTS_SCHEMA` to the module. This prevents errors related to unknown elements.

```tsx
import { NgModule, CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';

@NgModule({
  schemas: [
    CUSTOM_ELEMENTS_SCHEMA
  ]
})
```

## Template Creation

Create the form in your component's template.

```html
<form class="input-form" [formGroup]="searchForm" (ngSubmit)="submit($event)">
  <input type="text" formControlName="item" placeholder="Enter search term" tabindex="1" />
  <button type="submit">Search</button>
</form>
```

Breaking down the code above:

- `[formGroup]` specifies the name of the `FormGroup` instance defined in your component.
- `(ngSubmit)` binds the form's submit event to a function in your component.
- `formControlName` associates each input element with a specific `FormControl` within the `FormGroup`.

## Component Implementation

Now, let's implement the corresponding component logic.

```tsx
import { FormGroup, Validators, FormBuilder } from '@angular/forms';
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
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
      console.log('Form Valid:', this.searchForm.valid);
    });
  }

  submit(e: Event) {
    e.preventDefault(); // Prevent default form submission
    const { item } = this.searchForm.controls;
    console.log('Search Term:', item.value);
  }
}
```

First, import `FormGroup`, `Validators`, and `FormBuilder` from `@angular/forms`. `Validators` are optional and can be omitted if validation is handled in the template or not required.

```tsx
import { FormGroup, Validators, FormBuilder } from '@angular/forms';
```

Declare a variable for the `FormGroup` you'll use, matching the name specified in the template via `[formGroup]`.

```tsx
searchForm: FormGroup;
```

Inject the `FormBuilder` service via the constructor.

```tsx
constructor(
  private fb: FormBuilder
)
```

Define the `FormGroup` in the `constructor` (or `ngOnInit`).

```tsx
this.searchForm = this.fb.group({
  item: ['', [Validators.required]]
})
```

Specify the `formControlName` for each form control within the `FormGroup`. The first argument in the `fb.group` configuration is the initial value, and the second argument is an array of validators. If there's no initial value, use an empty string (`''`). If no validation is needed, you can provide an empty array.

Angular provides several built-in validators:

```tsx
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

Reference: [Angular Validators Documentation](https://angular.io/api/forms/Validators)

The `valueChanges` observable allows you to subscribe to changes in the form's values and react accordingly.

```tsx
this.searchForm.valueChanges.subscribe(observer => {
  console.log('Form Valid:', this.searchForm.valid);
});
```

Finally, implement the `submit` function to handle form submission, validation checks, data processing, and other actions.

```tsx
submit(e: Event) {
  e.preventDefault(); // Prevent default form submission
  const { item } = this.searchForm.controls;
  console.log('Search Term:', item.value);
}
```

That's it! You should now have a functional Angular form.