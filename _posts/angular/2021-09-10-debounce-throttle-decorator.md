---
title: "How to create decounce and throttle directives and decorators?"
date: 2021-09-10 17:54:00 +0900
comments: true
categories: javascript
tags: [decorator, directive, debounce, throttle, rxjs, typescript]
---

After searching extensively for examples of Debounce or Throttle Decorators suitable for Angular or TypeScript, I found myself needing to craft my own. I hope this post will be helpful to others on a similar quest.

Refer to my [previous post on decorators](https://ksrae.github.io/javascript/decorator/) for a detailed explanation. Here, we'll explore debounce and throttle, creating both directives and decorators and comparing their implementations.

Let's start by understanding debounce.

## Definition of Debounce

Debounce ensures that a function is executed only after a specified delay following the last time the event was emitted. If another event occurs during this waiting period, the previous event is canceled, and the delay restarts from the time of the new event.

Below are the implementations of debounce as a directive and a decorator.

### Creating a Debounce Directive

A directive can be easily created utilizing RxJS.

```tsx
@Directive({
	selector: '[debounce]'
})
export class DebounceDirective {
  @Output() debounceClick = new EventEmitter();

  constructor(private viewContainer: ViewContainerRef) {

  }

  ngOnInit() {
    fromEvent<MouseEvent>(this.viewContainer.element.nativeElement, 'click').pipe(debounceTime(500)).subscribe((e) => {
      this.debounceClick.emit(e);
    });
  }
}
```

To use the debounce Directive, your template should be structured as follows:

```html
{% raw %}
<button debounce (debounceClick)="submit($event)">
{% endraw %}
```

- Note: The above code uses a fixed `debounceTime`. Modification is necessary if you want to pass `debounceTime` as a parameter.

### Creating a Debounce Decorator

Let's create a debounce decorator. The principle involves canceling the previous event and executing the new event after a specified time (`ms`). The implementation is as follows:

```tsx
export function Debounce(ms: number = 0) {
  return (target : any, propertyKey : string, descriptor: PropertyDescriptor) => {
    const callback = descriptor.value;
    let runTime!: any;

    descriptor.value = (prop: unknown) => {
      if(runTime) {
          clearTimeout(runTime);
      }

      runTime = setTimeout(() => {
          callback(prop);
      }, ms);
    }
  }
}
```

To use this, your component should be implemented as follows:

```tsx
export class AppComponent {
	...
	@Debounce(1000)
	submit(e: Event) {
		...
	}
}
```

## Definition of Throttle

Throttle ensures that a function is executed only once during a specified period. For example, if `throttleTime` is 1000ms, the first request is executed, and any subsequent requests during that 1000ms period are ignored. After the `throttleTime` expires, a new request can be accepted and executed.

### Creating a Throttle Directive

Similar to debounce, throttle can be easily created using RxJS.

```tsx
@Directive({
	selector: '[throttle]'
})
export class ThrottleDirective {
  @Output() throttleClick = new EventEmitter();

  constructor(private viewContainer: ViewContainerRef) {

  }

  ngOnInit() {
    fromEvent<MouseEvent>(this.viewContainer.element.nativeElement, 'click').pipe(throttleTime(500)).subscribe((e) => {
      this.throttleClick.emit(e);
    });
  }
}
```

To utilize the throttle Directive, your template should resemble the following:

```html
{% raw %}
<button throttle (throttleClick)="submit($event)">
{% endraw %}
```

### Creating a Throttle Decorator

```tsx
export function Throttle(ms: number = 0) {
  return (target : any, propertyKey : string, descriptor: PropertyDescriptor) => {
    const callback = descriptor.value;
    let runTime!: any;
    let isrunning: boolean = false;

    descriptor.value = (prop: unknown) => {
      if(!isrunning) {
        callback(prop);
        clearTimeout(runTime);
        isrunning = true;
        runTime = setTimeout(() => {
            isrunning = false;
        }, ms);
      } 
    }
  }
}
```

To use this, your component should be implemented as follows:

```tsx
export class AppComponent {
	...
	@Throttle(1000)
	submit(e: Event) {
		...
	}
}
```

## Differences Between Debounce/Throttle Directives and Decorators

When implemented as a directive, the code can be concise, and since it's Angular code, it's easier to manage. However, when writing click events in the template, you must input directive calls and the Emit events set in the directive each time, which can be inconvenient.

On the other hand, when implemented as a decorator, the code becomes longer because it's written in TS, and management can be more challenging since it's not Angular code. However, using existing commands in the template simplifies the code, and you can directly control the time in the component function that handles the click event, making it more intuitive.

Ultimately, the choice depends on the developer's preference rather than one being objectively better than the other.