---
title: "How to Stop Angular Material Animation"
date: 2019-07-04 15:45:00 +0900
comments: true
categories: angular
tags: [material, animation]
---

When you need to disable animations in Angular Material components, whether it's for all animations or specific Material elements, there are several simple control methods you can use.

## Understanding Angular Material Animations

Angular Material comes with built-in animations that enhance the user experience. These animations are applied to various components such as:
- Tab transitions
- Dialog openings and closings
- Expansion panel toggles
- Menu appearances
- List item interactions

## Solution Methods

### Method 1: Disabling Animations for Specific Components

The simplest way to disable animations for a specific component is to add the `[@.disabled]="true"` attribute to the target element. This will immediately stop all animations for that component.

Here's a basic example:

```html
<mat-tab-group [@.disabled]="true">
  <mat-tab label="First"> Content 1 </mat-tab>
  <mat-tab label="Second"> Content 2 </mat-tab>
</mat-tab-group>
```

### Method 2: Conditional Animation Disabling

You can also dynamically control the animation state:

```html
<mat-tab-group [@.disabled]="isAnimationDisabled">
  <!-- tab content -->
</mat-tab-group>
```

```typescript
export class YourComponent {
  isAnimationDisabled = false;

  toggleAnimations() {
    this.isAnimationDisabled = !this.isAnimationDisabled;
  }
}
```

### Method 3: Global Animation Configuration
There are two different levels of animation control you should be aware of: <br/>
Angular Material-Specific Animations <br/>
To disable only Angular Material animations while keeping other Angular animations active, you can provide the configuration in your module:

```typescript
import { MAT_RIPPLE_GLOBAL_OPTIONS } from '@angular/material/core';

@NgModule({
  providers: [
    {
      provide: MAT_RIPPLE_GLOBAL_OPTIONS,
      useValue: { disabled: true }  // This only disables Material ripple animations
    }
  ]
})
export class AppModule { }
```

### Method 4: Standalone Components

For standalone components, which were introduced in Angular 14+, you have a couple of approaches to handle animations:

#### Option 1: Individual Standalone Component

```typescript
import { Component } from '@angular/core';
import { provideAnimations, provideNoopAnimations } from '@angular/platform-browser/animations';
import { MatTabsModule } from '@angular/material/tabs';

@Component({
  selector: 'app-my-standalone',
  standalone: true,
  imports: [MatTabsModule],
  providers: [
    // Use this to disable animations for this component
    provideNoopAnimations()
    // Or use this to enable animations
    // provideAnimations()
  ],
  template: `
    <mat-tab-group>
      <mat-tab label="First"> Content 1 </mat-tab>
      <mat-tab label="Second"> Content 2 </mat-tab>
    </mat-tab-group>
  `
})
export class MyStandaloneComponent {}
```

#### Option 2: Global Configuration for Standalone Application

In your `main.ts` file:

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { provideAnimations, provideNoopAnimations } from '@angular/platform-browser/animations';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    // Choose one of these:
    provideNoopAnimations(), // Disable all animations
    // provideAnimations()  // Enable animations
  ]
}).catch(err => console.error(err));
```

## Important Considerations

1. **Nested Components**: When working with nested Material components, be cautious about where you apply the animation disable attribute. For optimal performance and to avoid potential bugs:
   - Apply `[@.disabled]="true"` only to the top-level component
   - Avoid applying it to nested child components
   - Test thoroughly when disabling animations in complex component hierarchies

2. **Performance Impact**: While animations can enhance user experience, they might impact performance on lower-end devices. Consider providing an option for users to disable animations if performance is a concern.

3. **Accessibility**: Some users might prefer reduced animations for accessibility reasons. It's good practice to respect system-level animation settings:

```typescript
import { ANIMATION_MODULE_TYPE } from '@angular/platform-browser/animations';

@Component({
  // ...
  providers: [
    {
      provide: ANIMATION_MODULE_TYPE,
      useValue: 'NoopAnimations'
    }
  ]
})
```

## Debugging Tips

If animations aren't being disabled as expected, check for:
- Proper syntax in the attribute binding
- Conflicting animation settings in parent components
- Correct import of BrowserAnimationsModule or proper standalone configuration
- No overriding animation definitions in your component
- When using standalone components, ensure animation providers are correctly configured

## Conclusion

Disabling Angular Material animations is straightforward when you understand the available methods. Choose the approach that best fits your specific use case, whether it's disabling animations for individual components, implementing a global solution, or working with standalone components. Always consider the impact on user experience and accessibility when managing animations in your application.

Remember to test thoroughly, especially when working with nested components, to ensure the animation settings behave as expected across your entire application.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/%ED%8A%B9%EC%A0%95-Material-%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98-%EC%A4%91%EC%A7%80)
<br/>