---
title: "What is decorator?"
date: 2021-09-07 20:01:00 +0900
comments: true
categories: javascript
tags: [decorator, typescript]
---

A decorator is a way to implement instructions that need to be executed repeatedly before a function is executed, as a separate function, and then easily call it with a single line of code.

Generally, it is called by adding `@Decorator` before it. It can be applied to classes, functions, variables, and even function parameters, but you should be careful because each has different parameters.

Below, we will explore the types of decorators.
(The code is written in TypeScript.)

## Class Decorator

Written above the class name.
Generally, the decorator being called has no parameters, and only the function name is entered.

A class decorator function calls the constructor as an argument and can access the constructor and functions of that class.
However, it cannot access variables declared in the class.

If you want to adjust the change of the class from the outside by putting parameters in the decorator, there is a way to put a factory in the front end.
In other words, it is a method of writing a factory that receives the decorator instead and this factory calls the actual class decorator function.

```tsx
@testFactory(false)
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

function testFactory(setDefault: boolean) {
	return setDefault ? setDeaultValue : null;
}
```

In the example above, it is determined whether to specify a value when creating a class depending on the parameter value of `testFactory`. <br/>
If the value is `true`, the class has the values `name = 'SM6', price = 10000, color = 'black'` as soon as it is called.<br/>
If the value is `false`, the class has the values `name = '', price = 0, color = 'white'` when it is created.

## Method Decorator

Written above the function name.
A method decorator function can have three arguments, in the following order:

- `target`: The prototype of the class. It can access all functions of the class (including the constructor).
- `propName`: The key value of the corresponding method. The function name of the function with the decorator is entered in `string` format.
- `description`: Has the function's functionality.

The example below is an example of preventing access to a function using a method decorator.

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

`target` is the same as the argument value of the class decorator.
In the case of `propName`, it has `test02` to which the `editable` decorator is applied.
And the description has three functions, each of which is as follows:

- `writable`: Whether modification is possible. If `false`, you cannot modify the parameter value when calling the function.
- `enumerable`: Whether it is an enumeration type. If `false`, you cannot see the value when calling `Object.keys()`.
- `configurable`: Whether to apply `writable` and `enumerable`. If `false`, both cannot be applied arbitrarily.
- `value`: value (functional)

Note that `value` has the value of the function in the form of a function.

## Parameter Decorator

You can also set decorators for each parameter of a function.
In this case, it can also have three arguments, in the following order:

- `target`: The prototype of the class. It can access all functions of the class (including the constructor).
- `methodName`: The key value of the function including the parameter. The function name of the parameter with the decorator is entered in `string` format.
- `paramIndex`: Refers to the calling order of the parameters, and has a value of 0 when called first.

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

In the example above, `methodName` has `test` and `paramIndex` has a value of `0`.

Interestingly, you can also put a decorator on the parameters of the constructor. For example, if you put a decorator on `age` in the example above, it is like the example below.

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

The result is as follows:

```
methodName: undefined
paramIndex: 1
```

In other words, even though it was called in the constructor, the function name has an `undefined` value and the calling order is second.

### Use of Parameter Decorator

In the example, you can see that the parameter decorator has no return value.
So how can the parameter decorator be used?

To use this, you must explicitly access `reflect-metadata`.
Through this, you can modify the value of the parameter.

If you want to import `refrect-metadata`, you must add `emitDecoratorMetadata` in the options of `tsconfig.json`.

```json
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

This will not be covered in detail here, so please refer to the related site.

## Variable Decorator

Finally, you can put a decorator on a variable, which can have two arguments.

- `target`: The prototype of the class. It can access all functions of the class (including the constructor).
- `propName`: The key value of the corresponding variable. The variable name of the variable with the decorator is entered in `string` format.

The variable decorator can return a `PropertyDescriptor` type, which is the same as the description of the method decorator, as a return value.
The example below is a method of determining whether to make a variable `readonly` depending on the decorator's parameter.

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

## Reference

- [Typescript Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
- [Let's create a TypeScript Decorator](https://dparkjm.com/typescript-decorators)
- [IT Mining - Typescript Decorator](https://itmining.tistory.com/88)
- [Typescript's Decorator - 1. Class Decorator](https://partnerjun.tistory.com/61)
- [Typescript's Decorator - 2. Property Decorator, Simple Dependency Injection](https://partnerjun.tistory.com/62?category=705423)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Decorator-%EB%9E%80)
<br/>