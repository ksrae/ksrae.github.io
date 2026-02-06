---
title: Building Polymorphic Forms with Independent Components
date: 2025-12-17 12:00:00 +0900
comments: true
categories: angular
tags: [form, formarray, dynamic]
---

When building enterprise applications, we often encounter forms that need to change their entire structure based on user input. A classic example is a registration form that distinguishes between **"Locals"** (requires Resident ID) and **"Foreigners"** (requires Passport Number).

A common mistake is to handle all this logic in a single parent component. This leads to a massive, unmaintainable "God Component" where the parent knows too much about every specific subtype's validation logic.

In this post, I will demonstrate a **"Polymorphic Form Architecture"** where the parent acts only as a container (slot), and child components manage their own form logic independently.

## The Problem: The "God Component"

If the parent component creates the `FormGroup` for both Locals and Foreigners and passes it down to children, tight coupling occurs.

1. **Violation of Separation of Concerns:** The parent has to import validators for every possible sub-form.
2. **Maintenance Nightmare:** Adding a new type (e.g., "Corporate Member") requires modifying the parent's logic, template, and validation rules.

## The Solution: FormArray as a Dynamic Slot

To solve this, we use `FormArray` not as a list, but as a **single dynamic slot**.

1. **Child Component:** Creates its own `FormGroup` and emits it to the parent via `@Output`.
2. **Parent Component:** Listens to the event, clears its `FormArray` slot, and pushes the new `FormGroup` received from the child.
3. **Result:** The parent's `form.valid` status automatically syncs with the child's form status without the parent knowing the child's internal details.

### Step 1: The Independent Child Component

The child component is responsible for its own schema and validation. It notifies the parent whenever the form is ready.

```jsx
// local-form.component.ts
import { Component, inject, Output, EventEmitter, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';

@Component({
  selector: 'app-local-form',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <div [formGroup]="form" class="panel">
      <h4>🇰🇷 Local Resident Info</h4>
      <div class="field">
        <label>Resident Registration No.</label>
        <input formControlName="residentId" placeholder="000000-0000000" />
        @if (form.get('residentId')?.invalid && form.get('residentId')?.touched) {
          <small class="error">Invalid ID format.</small>
        }
      </div>
    </div>
  `
})
export class LocalFormComponent implements OnInit {
  private fb = inject(FormBuilder);
  
  // 1. Manage internal form group independently
  form: FormGroup = this.fb.group({
    residentId: ['', [Validators.required, Validators.pattern(/^\d{6}-\d{7}$/)]]
  });

  // 2. Emit the form instance to the parent
  @Output() formReady = new EventEmitter<FormGroup>();

  ngOnInit() {
    // Notify parent immediately upon initialization
    this.formReady.emit(this.form);
  }
}
```

### Step 2: The Parent "Slot Manager"

The parent component uses `FormArray` named `dynamicSection` to attach the child's form. It doesn't care what's inside the form, it just cares that it exists.

```jsx
// registration.component.ts
import { Component, inject } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators, FormArray, FormGroup } from '@angular/forms';
import { JsonPipe } from '@angular/common';
import { LocalFormComponent } from './local-form.component';
import { ForeignerFormComponent } from './foreigner-form.component';

@Component({
  selector: 'app-registration',
  standalone: true,
  imports: [ReactiveFormsModule, JsonPipe, LocalFormComponent, ForeignerFormComponent],
  templateUrl: './registration.component.html'
})
export class RegistrationComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group({
    name: ['', Validators.required],
    userType: ['local', Validators.required], // Switcher
    dynamicSection: this.fb.array([])         // The Slot
  });

  get dynamicSection(): FormArray {
    return this.form.controls.dynamicSection;
  }

  // 3. Handle the form provided by the child
  onChildFormReady(childForm: FormGroup) {
    this.dynamicSection.clear(); // Clear previous slot
    this.dynamicSection.push(childForm); // Attach new form
    
    // Now, this.form.valid includes the child form's validation status
  }

  onSubmit() {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }
    // Payload includes 'dynamicSection' containing child data
    console.log('Submission:', this.form.getRawValue());
  }
}
```

### Step 3: The Template with Control Flow

We use `@switch` to physically render the correct component. The `(formReady)` event binds them together.

```jsx
<form [formGroup]="form" (ngSubmit)="onSubmit()">
  
  <div class="section">
    <label>Name</label>
    <input formControlName="name" />
  </div>

  <div class="section">
    <label>User Type</label>
    <div class="radio-group">
      <label><input type="radio" formControlName="userType" value="local"> Local</label>
      <label><input type="radio" formControlName="userType" value="foreigner"> Foreigner</label>
    </div>
  </div>

  <hr />

  <div formArrayName="dynamicSection">
    @switch (form.value.userType) {
      @case ('local') {
        <app-local-form (formReady)="onChildFormReady($event)"></app-local-form>
      }
      @case ('foreigner') {
        <app-foreigner-form (formReady)="onChildFormReady($event)"></app-foreigner-form>
      }
    }
  </div>

  <button type="submit" [disabled]="form.invalid">Register</button>
</form>

<pre>{{ form.getRawValue() | json }}</pre>
```

## Conclusion

By treating `FormArray` as a container slot and letting child components manage their own `FormGroup`, we achieve true **component isolation**.

- **Scalability:** Adding a new form type is as simple as creating a new component and adding a `@case`.
- **Maintainability:** Local validation logic stays in `LocalFormComponent`.
- **Data Integrity:** The parent form always reflects the validity of the currently active child form.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EB%8F%85%EB%A6%BD%EC%84%B1%EC%9D%84-%EB%B3%B4%EC%9E%A5%ED%95%98%EB%8A%94-%EB%8B%A4%ED%98%95%EC%84%B1-%ED%8F%BCPolymorphic-Form-%EC%84%A4%EA%B3%84)
<br/>