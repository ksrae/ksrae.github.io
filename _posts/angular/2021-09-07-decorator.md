---
title: "Decorator 란? (What is decorator?)"
date: 2021-09-07 20:01:00 +0900
comments: true
categories: javascript
tags: [decorator, typescript]
---

함수가 실행 되기 전에 반복적으로 선행되어야 할 명령을 별도의 함수로 구현한 뒤 이를 한줄의 명령어로 쉽게 호출하는 방법 입니다.
일반적으로 @Decorator와 같이 앞에 @을 붙여서 호출하며, 클래스, 함수, 변수, 심지어 함수의 파라미터에도 적용할 수 있으나 각각의 적용하는 파라미터가 다르므로 유의하여야 합니다.

아래에서는 Decorator의 종류에 대해 알아보겠습니다.
(코드는 typescript에서 작성되었습니다.)

## Class Decorator
클래스명 상단에 작성합니다.
일반적으로 호출하는 Decorator에는 파라미터가 없고 함수명만 기입합니다.

class Decorator 함수는 생성자를 인수로 호출하며, 해당 클래스의 생성자 및 함수에 접근할 수 있습니다.
다만, 클래스에 선언된 변수에는 접근할 수 없습니다.

만일 Decorator에 파라미터를 넣어 클래스의 변화를 외부에서 조정하고 싶다면, Factory를 앞단에 두는 방법이 있습니다.
즉, Decorator를 대신 받아주는 factory를 작성하고, 이 factory가 실제 Class Decorator 함수를 호출하는 방법 입니다.

```tsx
@test01Factory(false)
export class Car {
  name: string;
	price: number;
	color: string;

	constructor(name?: string, price?: number, color?: string) {
		this.name = name ? name : '';
		this.price = price ? price : 0;
    this.color = color ? color : 'white';
	}
}

function setDeaultValue(constructorFn: any) {
	return <any>class extends constructorFn{
		name = 'SM6';
    price = 10000;
		color = 'black';
	}
}

function test01Factory(setDefault: boolean) {
	return setDefault ? setDeaultValue : null;
}
```

위의 예시에서는 test01Factory의 파라미터 값에 따라 클래스 생성시 값을 지정할지 여부를 결정합니다.
만일 값이 true라면 클래스는 호출되는 즉시 `name = 'SM6', price = 10000, color = 'black' ` 값을 가지고,
만일 값이 false라면 클래스는 생성 시 `name = '', price = 0, color = 'white' ` 값을 가지게 됩니다.



## Method Decorator
함수명 상단에 작성합니다.
Method Decorator함수는 3가지 인수를 가질 수 있으며 순서대로 다음과 같습니다.
- target: class의 prototype. 클래스의 모든 함수 (생성자 포함)에 접근할 수 있습니다.
- propName: 해당 method의 key 값이며, Decorator가 붙은 함수의 함수명이 string형태로 들어옵니다.
- description: 함수가 가지는 기능을 가집니다.

아래의 예시는 Method Decorator를 활용하여, 함수의 접근을 막는 예제입니다.

```tsx
class Person {
	constructor() {
		console.log('new Person()');
	}

	test01() {
		console.log('test01');
	}

	@editable(true)
	test02() {
		console.log('test02');
	}

}

function editable(isEditable: boolean) {
	return (target: any, propName: string, description: PropertyDescriptor) => {
    description.writable = isEditable;
	}
```
target은 Class Decorator의 인수 값과 동일합니다.
propName의 경우 여기에서는 editable Decorator가 적용된 `test02`를 갖습니다.
그리고, description에는 3가지 기능이 있는데 각각 다음과 같습니다.
- writable: 수정 가능 여부, false이면 함수를 호출할 때 파라미터 값을 수정할 수 없습니다.
- enumerable: 열거형인지 여부, false이면 Object.keys()를 호출할 때 값을 볼 수 없습니다.
- configurable: writable, enumerable 적용 여부, false이면 둘다 임의로 적용 불가능 합니다.
- value: 값 (함수형)

value는 함수의 값을 가지는데 함수형의 형태로 가지므로 유의하여야 합니다.


## Parameter Decorator
함수의 각각의 파라미터에도 Decorator를 설정할 수 있습니다.
이 경우에도 3가지 인수를 가질 수 있는데, 순서대로 다음과 같습니다.
- target: class의 prototype. 클래스의 모든 함수 (생성자 포함)에 접근할 수 있습니다.
- methodName: 해당 파라미터를 포함한 함수의 key 값이며, Decorator가 붙은 파라미터의 함수명이 string형태로 들어옵니다.
- paramIndex: 파라미터의 호출 순서를 의미하며, 첫번째로 호출되면 0 값을 갖습니다.


```tsx
function printInfo(target: any, methodName: string, paramIndex: number) {
	console.log('target', target);
	console.log('methodName', methodName);
	console.log('paramIndex', paramIndex);
}

class Person {
	private _name: string;
	private _age: number;

	constructor(name: string, age: number) {
		this._name = name;
		this._age = age;
	}

	test(@printInfo message: string) {
		console.log('message');
	}
}
```
위의 예제에서 methodName은 test를 가지며, paramIndex는 0값을 가집니다.

재미있게도 생성자의 파라미터에도 Decorator를 걸 수 있습니다. 예를 들어, 위의 예제의 age에 decorator를 걸면, 아래의 예제와 같으며,

```tsx
function printInfo(target: any, methodName: string, paramIndex: number) {
	console.log('target', target);
	console.log('methodName', methodName);
	console.log('paramIndex', paramIndex);
}

class Person {
	private _name: string;
	private _age: number;

	constructor(name: string, @printInfo age: number) {
		this._name = name;
		this._age = age;
	}

	test(message: string) {
		console.log('message');
	}
}
```

결과는 다음과 같습니다.
```
methodName: undefined
paramIndex: 1
```
즉, 생성자에서 호출하였으나 함수명은 undefined 값을 가지며, 호출 순서는 2번째를 갖습니다.

### Parameter Decorator의 활용
예제에서 보면 Parameter Decorator는 리턴 값이 없는 것을 볼 수 있습니다.
그러면 Parameter Decorator는 어떻게 활용할 수 있을까요?

이를 활용하기 위해서는 "reflect-metadata"에 명시적으로 접근해야 합니다.
이를 통해 파라미터의 값을 수정할 수 있습니다.

만일 "refrect-metadata"를 import하고자 한다면, tsconfig.json의 옵션에서 `emitDecoratorMetadata` 를 추가하여야 합니다.

```json
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

이에 대해서는 여기에서 자세히 다루지 않으므로 관련 사이트를 참고 하시기 바랍니다.


## Variable Decorator
마지막으로 변수에도 Decorator를 걸 수 있는데 2개의 인수를 가질 수 있습니다.
- target: class의 prototype. 클래스의 모든 함수 (생성자 포함)에 접근할 수 있습니다.
- propName: 해당 변수의 key 값이며, Decorator가 붙은 변수의 변수명이 string형태로 들어옵니다.

Variable Decorator는 리턴값으로 Method Decorator의 description과 같은 PropertyDescriptor 형을 리턴할 수 있습니다.
아래의 예제는 decorator의 파라미터에 따라 변수를 readonly로 만들지 여부를 결정하는 방법 입니다.

```tsx
function writable(isWritable: boolean) {
	return function(target:any, propName: string): any {
		return {
			writable: isWritable
		}
	}
}

class Person {
	@writable(true)
	name: string = 'Mark';

	constructor() {
		console.log('new Person()');
	}
}
```

## 참고 사이트
- [Typescript Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
- [TypeScript Decorator 직접 만들어보자](https://dparkjm.com/typescript-decorators)
- [IT 마이닝 - Typescript Decorator](https://itmining.tistory.com/88)
- [Typescript의 Decorator - 1. Class Decorator](https://partnerjun.tistory.com/61)
- [Typescript의 Decorator - 2. Property Decorator, 간단한 Dependency Injection](https://partnerjun.tistory.com/62?category=705423)

