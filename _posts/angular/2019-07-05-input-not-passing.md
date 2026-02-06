---
title: "@Input Failes Passing Data"
date: 2019-07-05 12:08:00 +0900
comments: true
categories: angular
tags: [input, ngonchange, subject, observable]
---

This post analyzes why @Input fails to pass data.

## Situation
When updating data while maintaining the page state (without routing), child components or directives passed through @Input are not functioning.

## Symptoms
- Even when forcibly sending identical data, it maintains the existing state and doesn't pass to child components or directives.
- No matter how many times you push to an Array, it doesn't pass to child components or directives.

## Solutions
1. Check if the same value was passed
@Input doesn't pass identical values. In other words, if the same value comes in multiple times, it ignores this and doesn't trigger events.

2. Verify if events are coming into ngOnChange(value: SimpleChanges)
If they are coming in, you might not be receiving them due to LifeCycle issues. You can handle it directly in ngOnChange, but note that ngOnChange is processed before ngOnInit.

3. If all else fails, passing through Service using Subject or Observable is the most reliable solution
This is the most reliable solution as it definitely passes data despite all the above symptoms.<br/>
Let me explain Angular's @Input data passing issues in more detail.<br/>

## What is Angular's @Input?
@Input is a decorator used in Angular to pass data from parent components to child components. You can think of it as similar to props in React.

### Problem Description
This issue occurs when updating data on the same page without page refresh or routing.

### Specific Symptoms
When trying to pass identical data:

```typescript
// Parent component
@Component({
  template: '<child-component [data]="someData"></child-component>'
})
class ParentComponent {
  someData = "Hello";
  
  updateData() {
    this.someData = "Hello"; // Reassigning the same value
  }
}
```

In this case, the child component won't detect the change.

#### When changing array data:

```typescript
// Parent component
@Component({
  template: '<child-component [items]="itemList"></child-component>'
})
class ParentComponent {
  itemList = [];
  
  addItem() {
    this.itemList.push('new item'); // Adding new item to array
  }
}
```

Simply pushing to an array might not be detected as a change.

### Solutions
1. Changing Reference Values
Create and pass a new reference even if the data is the same:

```typescript
updateData() {
  this.someData = new String("Hello"); // Creating new string object
  // Or for arrays
  this.itemList = [...this.itemList, 'new item']; // Creating new array
}
```

2. Using ngOnChanges
Directly detect changes in the child component:

```typescript
// Child component
@Component({
    ...
})
class ChildComponent implements OnChanges {
  @Input() data: any;

  ngOnChanges(changes: SimpleChanges) {
    if (changes['data']) {
      // Code to execute whenever data changes
      console.log('Data changed:', changes['data'].currentValue);
    }
  }
}
```

3. State Management through Services
Instead of using Input, you can pass data streams through services.<br/>
Note that all Components using this service will share the same data.

```typescript
// Data service
@Injectable({
  providedIn: 'root'
})
class DataService {
  private dataSubject = new BehaviorSubject<any>(null);
  data$ = this.dataSubject.asObservable();

  updateData(newData: any) {
    this.dataSubject.next(newData);
  }
}

// Child component
@Component({
    ...
})
class ChildComponent implements OnInit {
  constructor(private dataService: DataService) {}

  ngOnInit() {
    this.dataService.data$.subscribe(data => {
      // Code to execute whenever data changes
    });
  }
}
```

## Additional Tips
When passing objects or arrays, it's always good to maintain immutability.<br/>
Consider using state management libraries like NgRx or RxJS if you need complex data flows.<br/>
Using ChangeDetectionStrategy.OnPush improves performance but requires more attention to data change detection.<br/>

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Input%EC%9D%B4-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%A5%BC-%EC%A0%84%EB%8B%AC%ED%95%98%EC%A7%80-%EB%AA%BB%ED%95%98%EB%8A%94-%EC%9B%90%EC%9D%B8)
<br/>