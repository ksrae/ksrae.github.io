---
title: "Customizing Form Group Error"
date: 2019-07-12 17:31:00 +0900
comments: true
categories: angular
tags: [form, formcontrol, formgroup, error, custom]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%ED%8F%BC-%EA%B7%B8%EB%A3%B9-%EC%97%90%EB%9F%AC%EC%B2%98%EB%A6%AC-%EC%BB%A4%EC%8A%A4%ED%84%B0%EB%A7%88%EC%9D%B4%EC%A7%95)
<br/>

When working with Angular's `FormControl` or `FormGroup` in conjunction with Angular Material, customizing error handling often becomes necessary. For instance, you might need to display an error message if a mandatory agreement checkbox is not checked.

While you could implement separate validation and CSS styling for these messages, this guide explores how to customize error handling while retaining the default settings as much as possible. After exploring various approaches, the solution outlined below seems to be the most effective.

## Error Handling in FormControl

To trigger an error in a `FormControl`, use the `setError` method on the control and check for the error using `hasError` within the `mat-error` element.

```html
<mat-form-field>
  <input [formControl]="testControl" #testControl type="text" />
  <mat-error *ngIf="testControl.hasError('testError')">Error Message</mat-error>
</mat-form-field>
```

```tsx
// Triggering an Error
this.testControl.setError({ testError: true });
```

## Error Handling in FormGroup

```html
<form [formGroup]="testForm">
  <mat-form-field class="full-width">
    <input matInput type="text" formControlName="checkControl" />
    <mat-error>Please verify the authentication code.</mat-error>
  </mat-form-field>
</form>
```

```tsx
// Component
...
ngOnInit() {
  this.testForm = this.fb.group({
    checkControl: ['', []]
  });
}

customError() {
  this.testForm.get('checkControl').markAsTouched();
  this.testForm.get('checkControl').setErrors({ incorrect: true });
}
```

## Important Considerations

- Errors are only triggered after the control has been touched. Therefore, you must call `markAsTouched()` before setting the error using `setErrors()`. Otherwise, the error will not be applied.
- To clear an error, use `setError(null)` or `setError({ incorrect: false })`.
- A `FormGroup` is considered valid only when all its controls have a null error state.
- An alternative approach involving `this.testForm.get('checkControl').updateValueAndValidity({onlySelf: true, emitEvent: false});` was tested but did not function as expected. If you find this method to be effective, please share your insights in the comments.