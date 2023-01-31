---
title: "typeof, instanceof, keyof and in의 차이점(difference among typeof, instanceof, keyof and in)"
date: 2023-01-30 11:07:00 +0900
comments: true
categories: angular
tags: [typescript, instanceof, typeof, keyof, in]
---

> TypeScript에서 typeof, instanceof, keyof, in 연산자는 각각 다음과 같은 역할을 합니다.

## typeof 연산자
typeof 연산자는 변수 또는 값의 타입을 반환하는 데 사용됩니다.

```javascript
let myNumber = 123;
console.log(typeof myNumber); // 출력: number

let myString = "hello";
console.log(typeof myString); // 출력: string
```

위 예제에서 myNumber 변수의 타입은 number이므로 number가 출력됩니다. 또한 myString 변수의 타입은 string이므로 string이 출력됩니다.

### 확인 가능한 자료형
확인 가능한 자료형은 다음과 같습니다.

- "undefined": undefined
- "boolean": boolean
- "number": 일반적인 숫자형 Number
- "bigint": Number보다 큰 숫자형
- "string": string
- "symbol": Symbol (ECMAScript 2015에서 추가됨)
- "function": 함수형
- "object": 다른 모든 객체

> number는 64비트 부동소수점 포멧입니다. 자세한 내용은 [링크](https://en.wikipedia.org/wiki/Double-precision_floating-point_format)를 확인하세요.

> bigint는 [Arbitrary-precision arithmetic](https://en.wikipedia.org/wiki/Arbitrary-precision_arithmetic) 를 참고하세요.

## instanceof 연산자
instanceof 연산자는 객체가 특정 클래스의 인스턴스인지 확인하는 데 사용됩니다.<br/>
<br/>
javascript에서는 함수의 prototype이 존재하는지 여부를 확인하며, <br/>
typescript에서는 class의 데이터 형인지 여부를 확인합니다.<br/>
<br/>
boolean 형을 리턴합니다.<br/>

```js
function Car(make, model, year) {
this.make = make;
this.model = model;
this.year = year;
}

const auto = new Car('Honda', 'Accord', 1998);
console.log(auto instanceof Car);
Car.prototype = {};
console.log(auto instanceof Car);
```

javascript에서는 auto가 Car의 prototype을 가졌으므로 첫번째 console.log의 값은 true가 됩니다.<br/>
그러나 만일 두번째 console.log의 케이스와 같이 auto를 선언한 후에 Car를 다른 값으로 변경했다면, 값은 false가 됩니다. <br/>

```tsx
class User {
  name: string;
  constructor(*name*: string) {
    this.name = name;
  }
}

const person = new User('username');
console.log(person instanceof User);
```

person은 User Class 형을 가지므로 true가 됩니다.


### type guard

두 가지 이상의 class에 동일한 interface가 존재할 때 instanceof 를 통해 이 interface가 정확히 어떠한 class의 것인지를 지정해줍니다.

```tsx
class Foo{}
class Bar extends Foo{}
class Bas{}

var bar = new Bar();
console.log(bar instanceof Bar); // true
console.log(bar instanceof Foo); // true
console.log(bar instanceof Object); // true
console.log(bar instanceof Bas); // false

```

위의 코드에서 bar는 Bar 형으로 선언되어 있으며, Bar는 Foo를 상속하고 있습니다.<br/>
이 때 bar는 console.log를 통해 Foo 또는 Bar일 때 true값을 리턴합니다.<br/><br/>
type guard는 여기에서 하나 더 나아가 bar가 Foo타입인지 Bar타입인지 조건을 보고 그에 맞는 행동만을 수행하도록 합니다.<br/><br/>
특히 generic 타입의 경우 이렇게 하지 않으면 타입 오류가 발생하여 에러가 나므로 함수를 수행하기 전에 반드시 type guard를 통해 타입을 명확히 하도록 해야 합니다.


## keyof 연산자
keyof 키워드는 객체 타입의 프로퍼티 이름의 타입을 정의할 때 사용합니다.

### in Json

```javascript
let myObject = { name: "John", age: 30 };
let myObjectKey: keyof typeof myObject;
// name, age
```

위 예제에서 myObjectKey 변수는 myObject 객체의 프로퍼티 이름(name과 age)의 타입을 나타냅니다.<br/>
keyof 키워드를 사용하면 객체 타입의 프로퍼티 이름을 타입으로 지정하여 객체 타입의 프로퍼티 이름에 대한 타입 안정성을 높일 수 있습니다.<br/><br/>
이 때, 유의할 점은 typeof와 함께 사용해야 원하는 값을 얻을 수 있다는 점입니다.<br/>
type 변수에서는 조금 더 간단하게 작성할 수 있습니다.<br/>


### in Type
keyof 키워드를 Type 변수에서도 사용할 수 있습니다.

```typescript
type User = { name: string, age: number, address: string };
let userKey: keyof User;
// name, age, address
```

위 예제에서 userKey 변수는 User 객체의 프로퍼티 이름(name, age, address)의 타입을 나타냅니다.<br/>
keyof 키워드를 사용하면 Type 변수의 프로퍼티 이름을 타입으로 지정하여 객체 타입의 프로퍼티 이름에 대한 타입 안정성을 높일 수 있습니다.<br/>


## in 연산자
in 연산자는 Object 또는 Array가 특정 프로퍼티를 가지고 있는지 확인하는 데 사용됩니다.<br/>
일반적으로 json값에서 해당 키가 존재하는지 또는 enum 값에서 존재하는 값인지를 확인하기 위해 사용합니다.<br/>

```javascript
let myObject = { name: "John", age: 30 };
console.log("name" in myObject); // 출력: true
console.log("email" in myObject); // 출력: false
```

위 예제에서 myObject 객체는 name 프로퍼티를 가지고 있으므로 true가 출력됩니다. <br/>
그러나 email 프로퍼티는 가지고 있지 않으므로 false가 출력됩니다.


### index in Array

배열에서도 사용할 수 있으나 배열에서는 값을 찾을 수 없고, index만 확인할 수 있습니다.<br/>
다음의 예를 참고하세요.<br/>

```tsx
let count = ['one', 'two', 'three', 'four'];
0 in count         // true
(1 + 2) in count   // true
6 in count         // false
"one" in count     // false, 배열의 내용이 아닌, 인덱스 값이 들어가야 합니다.
"length" in count  // true, length는 배열의 속성이기 때문입니다.
```

### key in Json

json에서는 key가 존재하는지 확인할 때 사용합니다.<br/>

```tsx
let user = {
  name: 'username',
  age: 10
}
console.log('name' in user) // true
```

### not working in Interface
interface 로 선언한 경우 interface에 존재하는 객체인지 확인하려고 시도하면 에러가 발생합니다.<br/>
이는 interface는 변수에 형식만 참조하도록 제공하는 역할을 할 뿐이고 실제 값을 가지고 있지는 않기 때문입니다.<br/>

```tsx
interface User {
  name: string;
  age: number;
}

let anotherUser: User = {
  name: 'anotheruser',
  age: 20
}

console.log('name' in User) // error
console.log('name' in anotherUser) // true
```

정리하자면, TypeScript에서 typeof, instanceof, in 연산자는 각각 변수 또는 값의 타입, 객체의 인스턴스 여부, 객체의 프로퍼티 여부를 확인하는 데 사용됩니다. <br/>
이 연산자들을 적절히 사용하면 프로그램의 타입 체크와 객체의 프로퍼티 여부를 효율적으로 확인할 수 있습니다.<br/>

## 참고 사이트

- [MDN-instanceof](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/instanceof)
- [typescript MyObject.instanceOf()](https://stackoverflow.com/questions/24705631/typescript-myobject-instanceof)
- [TypeScript 핸드북 10 - 고급 타입](https://infoscis.github.io/2017/06/19/TypeScript-handbook-advanced-types/)