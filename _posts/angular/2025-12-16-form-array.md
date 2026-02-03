---
title: Dynamic Forms - FormArray Schemas based on Selection
date: 2025-12-16 12:00:00 +0900
comments: true
categories: angular
tags: [form, formarray]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Angular-FormArray-%ED%8F%BC-%EA%B5%AC%EC%A1%B0-%EB%8F%99%EC%A0%81%EC%9C%BC%EB%A1%9C-%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0)
<br/>

When building complex registration forms, we often encounter scenarios where the form structure must change entirely based on a user's selection. A classic example is a sign-up page distinguishing between "Locals" and "Foreigners."

Locals might need to provide a Resident Registration Number, while Foreigners need to provide a Passport Number and Visa Type. Handling this with static `*ngIf` checks can lead to validation nightmares where hidden fields remain invalid.

In this post, I will demonstrate how to dynamically manage these schema changes using `FormArray` and Angular's latest features.

## The Problem

We need a registration form with the following requirements:

1. A Radio Button to select **Local** or **Foreigner**.
2. **Local selected:** The form must dynamically generate fields for 'Resident ID'.
3. **Foreigner selected:** The form must dynamically generate fields for 'Passport Number' and 'Nationality'.
4. Switching between types must clear previous data and validations immediately.
5. Submission should only include valid data for the selected type.

## The Solution: Dynamic FormArray Re-generation

Instead of hiding/showing controls visually, we will physically `clear()` and `push()` new FormGroups into a `FormArray` whenever the radio value changes. This ensures the form's validity state is always accurate.

### Step 1: Component Logic (TypeScript)

We use `toSignal` (optional but recommended in modern Angular) or simple `valueChanges` subscription to trigger the schema updates.

```jsx
// signup-form.component.ts
import { Component, inject, OnInit, OnDestroy } from '@angular/core';
import { FormArray, FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';
import { JsonPipe } from '@angular/common';
import { Subject, takeUntil } from 'rxjs';

@Component({
  selector: 'app-signup-form',
  standalone: true,
  imports: [ReactiveFormsModule, JsonPipe],
  templateUrl: './signup-form.component.html',
  styleUrl: './signup-form.component.scss'
})
export class SignupFormComponent implements OnInit, OnDestroy {
  private fb = inject(FormBuilder);
  private destroy$ = new Subject<void>();

  // 1. Define the Root Form
  form = this.fb.group({
    name: ['', Validators.required],
    userType: ['local', Validators.required], // Default to Local
    // We use FormArray to hold the dynamic section
    additionalInfo: this.fb.array([])
  });

  get additionalInfo(): FormArray {
    return this.form.controls.additionalInfo as FormArray;
  }

  ngOnInit() {
    // Initialize the form based on default value
    this.updateFormSchema('local');

    // 2. Listen to Radio Button Changes
    this.form.controls.userType.valueChanges
      .pipe(takeUntil(this.destroy$))
      .subscribe((type) => {
        if (type) {
          this.updateFormSchema(type);
        }
      });
  }

  // 3. Dynamic Schema Switching Logic
  private updateFormSchema(type: string) {
    this.additionalInfo.clear(); // Clear existing controls

    if (type === 'local') {
      // Schema for Locals: Resident ID
      const localGroup = this.fb.group({
        residentId: ['', [Validators.required, Validators.pattern(/^\d{6}-\d{7}$/)]]
      });
      this.additionalInfo.push(localGroup);

    } else if (type === 'foreigner') {
      // Schema for Foreigners: Passport & Nationality
      const foreignerGroup = this.fb.group({
        passportNum: ['', Validators.required],
        nationality: ['', Validators.required]
      });
      this.additionalInfo.push(foreignerGroup);
    }
  }

  onSubmit() {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }
    // Only the currently active fields are in the payload
    console.log('Submission:', this.form.getRawValue());
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### Step 2: Template with Control Flow

We use Angular's `@switch` or `@if` to render the correct UI for the controls currently inside the `FormArray`. Since the `FormArray` is the source of truth, we iterate through it.

```jsx
<form [formGroup]="form" (ngSubmit)="onSubmit()">
  
  <div class="section">
    <label>Name</label>
    <input formControlName="name" placeholder="John Doe">
  </div>

  <div class="section">
    <label>User Type:</label>
    <div class="radio-group">
      <label>
        <input type="radio" formControlName="userType" value="local"> Local (Korean)
      </label>
      <label>
        <input type="radio" formControlName="userType" value="foreigner"> Foreigner
      </label>
    </div>
  </div>

  <hr/>

  <div formArrayName="additionalInfo">
    @for (group of additionalInfo.controls; track group; let i = $index) {
      <div [formGroupName]="i" class="dynamic-panel">
        
        @if (form.value.userType === 'local') {
          <h4>Local Resident Info</h4>
          <div class="field">
            <label>Resident Registration No.</label>
            <input formControlName="residentId" placeholder="000000-0000000">
            @if (group.get('residentId')?.invalid && group.get('residentId')?.touched) {
              <small class="error">Valid ID is required.</small>
            }
          </div>

        } @else {
          <h4>Foreigner Visa Info</h4>
          <div class="field">
            <label>Passport Number</label>
            <input formControlName="passportNum" placeholder="M12345678">
            @if (group.get('passportNum')?.invalid && group.get('passportNum')?.touched) {
              <small class="error">Passport is required.</small>
            }
          </div>
          <div class="field">
            <label>Nationality</label>
            <input formControlName="nationality" placeholder="e.g. USA">
          </div>
        }

      </div>
    }
  </div>

  <button type="submit" [disabled]="form.invalid">Submit Registration</button>
</form>

<pre>{{ form.getRawValue() | json }}</pre>
```

### Step 3: Why this is safer?

If we simply hid the Foreigner inputs with CSS or standard `@if` while keeping them in the FormGroup, the `Validators.required` on the passport field would prevent a "Local" user from submitting the form.
By using `FormArray.clear()` and `push()`, we ensure that **only the controls visible to the user exist in the form model**.

## Conclusion

Using `FormArray` isn't just for simple lists of identical items. It acts as a powerful container for dynamic form sections that can be swapped out entirely based on business logic.

This approach ensures your form validation state always matches the UI, preventing common bugs in dynamic forms.
