---
title: "interface에서 getter, setter 생성하기(How to create getter and setter in interface?)"
date: 2022-04-29 13:13:00 +0900
comments: true
categories: typescript
tags: [class, interface, getter, setter]
---

class의 getter와 setter에 대해 알아보고, 이를 interface에서 구현합니다. <br/>

## getter와 setter의 정의
일반적으로 class는 변수를 private으로 지정하고, 함수를 통해 변수의 값을 설정하거나 가져오는데,<br/>
이 때, 해당 함수의 앞에 get과 set을 선언하며 이를 getter와 setter라고 합니다.<br/>
<br/>
코드로 보면 다음과 같습니다.<br/>

```tsx
class Person {
  private _name: string;

  constructor() {
  }

  get name(): string {
    return this._name;
  }

  set name(value: string) {
    this._name = value;
  }
}

```

Person class의 _name 변수의 값을 설정하거나 가져오려면 get name과 set name을 사용합니다.<br/>

## 일반함수로 getter, setter를 구현할 수 있지 않을까?
여기에서 의문이 드는 것은 일반 함수로도 다음과 같이 getter와 setter를 구현할 수 있지 않을까 하는 점입니다.<br/>

```tsx
class Person {
  private _name: string;

  constructor() {
  }

  getName(): string {
    return this._name;
  }

  setName(value: string): void {
    this._name = value;
  }
}

```

위의 클래스는 getter와 setter 대신 일반 함수인 getName()과 setName()을 작성하여 변수에 접근하는 것을 볼 수 있으며, 아무런 문제 없이 동작합니다.<br/>
그러면 과연 일반 함수와 getter, setter의 차이는 무엇일까요?<br/>
<br/>

### 1. getter와 setter는 동일한 함수이 름을 사용할 수 있다.
가장 큰 특징은 getter와 setter는 같은 함수 이름을 사용할 수 있다는 것입니다. <br/>
사실 함수의 이름은 어떤 것이든 상관없지만, 일반적으로는 변수의 이름과 같은 것을 사용합니다.<br/>

### 2. getter와 setter는 규칙을 반드시 따라야 한다.
일반 함수로 구현했을 때 다음과 같이 작성하여도 문제가 없습니다.<br/>

```tsx
class Person {
  private _name: string;

  constructor() {
  }

  getName(postfix: string): string {
    return `${this._name}${postfix}`;
  }

  setName(value: string): string {
    this._name = value;
    return this._name;
  }
}

// 문제 없이 작동합니다.
```

만일 getter와 setter에서 위와 같이 구현하면 어떻게 될까요?<br/>

```tsx
class Person {
  private _name: string;

  constructor() {
  }

  get name(postfix: string): string {
    return `${this._name}${postfix}`;
  }

  set name(value: string): string {
    this._name = value;
    return this._name;
  }
}

// error가 발생합니다.

```

getter와 setter는 반드시 규칙을 따라야 하며, 이를 지키지 않을 경우 에러를 발생시킵니다. <br/>
즉, 휴먼 에러를 방지하며 목적에 맞는 정확한 구현을 요구합니다.<br/>
규칙은 다음과 같습니다.<br/>
<br/>
- getter는 함수의 인자를 받을 수 없고, 반드시 리턴 값을 가져야 합니다.
- setter는 반드시 함수의 인자를 받아야 하며, 리턴 값을 가질 수 없습니다.


### 3. getter와 setter는 호출 시 ()를 생략할 수 있다.
외부에서 클래스의 getter와 setter에 접근할 때 함수의 형태를 생략할 수 있습니다. <br/>
만일 위와 같은 Person class가 존재한다고 할 때, Person 함수의 getter와 setter를 사용하는 코드를 작성하면 다음과 같습니다.<br/>

``` tsx
  class Main() {
    person = new Person();

    constructor() {
      person.name = 'John'; // setter
      let name = person.name; // getter
    }

  }
```

위의 예제를 보면 마치 Person class의 name 변수에 접근하는 것처럼 사용할 수 있는데 이는 getter와 setter으로만 가능하며, 일반 변수는 이와 같이 사용할 수 없습니다.<br/>


## interface에서 변수와 함수 구현하기
최종 목표인 interface에서 getter와 setter 구현하는 방법을 알아보기전에 변수와 함수를 어떻게 구현하는지를 먼저 알아보겠습니다.<br/>

### 변수 구현하기
변수를 구현하는 방법은 다음과 같습니다.<br/>

```tsx
interface Person {
  name: string;
  age: number;
}
```

### 함수 구현하기
함수를 구현하는 방법은 다음과 같습니다.

```tsx
interface IPerson {
  getName: (postfix: string) => string;
  setName: (value: string) => string;
}
```

### getter와 setter 구현하기
사실 interface는 getter와 setter를 지원하지 않으므로, 다음의 코드는 에러가 발생합니다.

```tsx
interface IPerson {
  get name(): string;
  set name(value: string);
}


// 이를 클래스에 적용하면 다음과 같은 에러가 발생합니다.
// src/app/app.component.ts: - error TS1131: Property or signature expected.
// get name(): string;
```

그러면 어떻게 적용할 수 있을까요?<br/>
정답은 아닐 수 있으나 getter와 setter에 대한 이해의 폭을 넓히면 쉽게 적용할 수 있습니다.<br/>
<br/>
getter와 setter의 목적은 결국 private 함수에 접근하기 위해서 사용하는 함수이며, 이를 반대로 생각하면 public 변수인 경우에는 getter와 setter가 필요하지 않습니다.<br/>
따라서, interface에서 public 변수를 선언해두면 getter와 setter 기능을 모두 포함한 private 변수를 구현하는 것과 동일한 효과가 있으며, 컴파일러도 이를 이해하여 에러를 발생히키지 않습니다.<br/>
<br/>
즉, Person 클래스는 다음과 같이 구현할 수 있습니다. <br/>

```tsx
interface IPerson {
  name: string;
}
```

맞는지 검증하기 위해 Person 클래스에 interface를 적용해 봅시다.<br/>

```tsx
class Person implements IPerson {
  private _name: string;

  constructor(name: string) {
    this._name = name;
  }

  get name(): string {
    return this._name;
  }

  set name(value: string) {
    this._name = value;
  }
}
```

실행하면 interface에서 선언한 name 변수가 getter와 setter를 대신하고 있음을 확인할 수 있습니다.<br/>
<br/>
<br/>
이 방법은 하나의 방안으로, 더 정확하고, 좋은 방법이 있다면 제보 부탁 드립니다.<br/>

