---
title: "A Guide to ControlValueAccessor for Custom Angular Form Controls"
date: 2025-01-16 12:46:00 +0900
comments: true
categories: angular
tags: [controlvalueaccessor, forms, reactiveforms]
---

When developing large-scale, complex applications with Angular, it's common to build a shared library of UI components to maintain consistent design standards. Form controls like input and select are core elements that are reused throughout the application.<br/>
However, the traditional approach of creating custom form controls using only @Input and @Output decorators can lead to several issues. For instance, it becomes cumbersome for the parent component to track the control's state (touched, dirty, valid, etc.) or to seamlessly integrate with Angular's form validation features.<br/>
To solve these problems, Angular provides a powerful interface called ControlValueAccessor. It acts as a bridge that allows our custom form controls to interact naturally with Angular's FormControl.<br/>

## What is ControlValueAccessor?
ControlValueAccessor is an interface that integrates a custom component into the Angular forms system, making it behave like a native DOM element. By implementing this interface, our component can easily use directives like formControlName and [(ngModel)].<br/>

The interface defines four key methods:

- writeValue(obj: any): void: Called by the Angular forms API to write a model value to the view (our custom control). In other words, it updates the UI of the custom component when the FormControl's value changes.
- registerOnChange(fn: any): void: Registers a callback function (fn) to be called when the value in the view (custom control) changes. Invoking this function passes the updated value to the FormControl.
- registerOnTouched(fn: any): void: Registers a callback function (fn) to be called when the user "touches" the control (e.g., on a blur event). This allows us to manage the control's touched state.
- setDisabledState?(isDisabled: boolean): void: Called when the FormControl's disabled state changes. We can implement this method to visually disable the UI of our custom control.

## The Limitation of the Traditional Approach (@Input / @Output)
Before using ControlValueAccessor, let's highlight the issues with the typical @Input/@Output method.

```TypeScript
// Parent Component TS (standalone)
import { Component } from '@angular/core';
import { ColorPickerTraditionalComponent } from './color-picker-traditional.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ColorPickerTraditionalComponent],
  template: `
    <app-color-picker-traditional
      [initialColor]="selectedColor"
      (colorChange)="onColorChange($event)">
    </app-color-picker-traditional>

    <!-- Additional logic is needed to check for touched or dirty states -->
    @if (isTouched) {
      <p>The control has been touched.</p>
    }
  `,
})
export class AppComponent {
  selectedColor = '#FFFFFF';
  isTouched = false;

  onColorChange(color: string) {
    this.selectedColor = color;
    this.isTouched = true; // State must be managed manually
  }
}
```

While this approach works for simple value passing, it requires the parent component to always listen for the colorChange event and involves writing extra code to track form states like touched or valid.

## Improving with ControlValueAccessor
Now, let's build the ColorPicker component using ControlValueAccessor. The goal is to allow the parent component to manage its state and value using a formControl, just like a standard input element.

### tep 1: Basic Component Setup
First, we need to register the component as a ControlValueAccessor in its providers. We use the NG_VALUE_ACCESSOR token and forwardRef for this.

```TypeScript
// color-picker.component.ts
import { Component, forwardRef, Input, signal } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Component({
  selector: 'app-color-picker',
  standalone: true,
  imports: [], // @for is built-in, no import needed
  templateUrl: './color-picker.component.html',
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => ColorPickerComponent),
      multi: true,
    },
  ],
})
export class ColorPickerComponent implements ControlValueAccessor {
  @Input() colors: string[] = [];

  // Manage internal state with Signals
  value = signal('');
  disabled = signal(false);
  
  // CVA callback functions
  onChange: (value: string) => void = () => {};
  onTouched: () => void = () => {};

  // 1. FormControl -> View
  writeValue(value: string): void {
    this.value.set(value);
  }

  // 2. View -> FormControl (Register change callback)
  registerOnChange(fn: any): void {
    this.onChange = fn;
  }

  // 3. View -> FormControl (Register touched callback)
  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }
  
  // 4. Called when the FormControl's disabled state changes
  setDisabledState(isDisabled: boolean): void {
    this.disabled.set(isDisabled);
  }

  // Method called when the user selects a color
  selectColor(color: string) {
    if (!this.disabled()) {
      this.value.set(color);
      this.onChange(this.value()); // Notify the FormControl of the change
      this.onTouched();          // Notify that the control has been touched
    }
  }
}
```

### Step 2: Creating the Component Template

```Html
<!-- color-picker.component.html -->
<div class="color-palette">
  @for (color of colors; track color) {
    <span
      class="color-box"
      [class.selected]="color === value()"
      [class.disabled]="disabled()"
      [style.background-color]="color"
      (click)="selectColor(color)">
    </span>
  }
</div>
```

### Step 3: Using it in the Parent Component
The parent component will also be standalone, importing ReactiveFormsModule and our ColorPickerComponent into its imports array.

```TypeScript
// app.component.ts
import { Component } from '@angular/core';
import { FormControl, FormGroup, ReactiveFormsModule } from '@angular/forms';
import { JsonPipe } from '@angular/common';
import { ColorPickerComponent } from './color-picker/color-picker.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ReactiveFormsModule, JsonPipe, ColorPickerComponent],
  templateUrl: './app.component.html',
})
export class AppComponent {
  availableColors = ['#e63946', '#f1faee', '#a8dadc', '#457b9d', '#1d3557'];

  colorForm = new FormGroup({
    favoriteColor: new FormControl('#a8dadc'), // Set initial value
  });
}
```


```Html
<!-- app.component.html -->
<form [formGroup]="colorForm">
  <h3>Choose your favorite color:</h3>
  <app-color-picker
    formControlName="favoriteColor"
    [colors]="availableColors">
  </app-color-picker>

  <hr>

  <h4>Form Control State:</h4>
  <pre>Value: {{ colorForm.get('favoriteColor')?.value | json }}</pre>
  <pre>Touched: {{ colorForm.get('favoriteColor')?.touched | json }}</pre>
  <pre>Dirty: {{ colorForm.get('favoriteColor')?.dirty | json }}</pre>
</form>
```

As a result, the parent component's code has become much cleaner. Thanks to ControlValueAccessor, app-color-picker is now a reusable component that fully supports all features of the Angular forms system.

## Conclusion
ControlValueAccessor is the standard and powerful way to create reusable custom form controls in Angular. While the initial setup might seem a bit complex, once you understand the structure, you gain significant benefits, including seamless integration with the Angular forms system, cleaner code, and easier maintainability.<br/>
If you need to build complex form controls, it is highly recommended to use ControlValueAccessor instead of the @Input/@Output combination.

# Reference
[ontrol Value Accessor](https://medium.com/@tapaswim/control-value-accessor-f0b85062d79)