---
title: "Enum 총정리"
date: 2019-07-09 15:40:00 +0900
comments: true
categories: angular
tags: [enum]
---



#### 1. 정의

Enum은 상수 값의 의미있는 정의라고 볼 수 있습니다.
즉, Yes, No를 처리하기 쉽게 

```ts
let response = 1;
let fail = 0;
```

```ts
const YES = 1;
const NO = 0;

let response = YES;
let fail = NO;
```

으로 선언하여 사용하는 것이 더 코드를 이해하기 쉽게 만들 수 있다는 것입니다.

Enum은 이러한 const를 유사한 것들로 묶어서 관리하는 const 세트로 보면 이해하기 쉬울 것입니다.

```ts
enum Response {
  No,
  Yes
}

let response = Response.YES;
let fail = Response.No;
```

위의 예시에서 const yes와 const no는 각각 독립적이어서 연관성이 없으나 enum으로 묶으면 Response라는 하나의 그룹에 속하여 연관성을 갖게 됩니다.

또한 Enum이 상수와 다른 점은 여러가지 다양한 방법으로 활용이 가능하다는 것입니다.
이번 시간은 이러한 특징을 알아보도록 하겠습니다.


#### 1. 숫자형 Enum

```ts
enum Type {
	INACTIVE,
	ACTIVE,
	ALL
}
```

아무것도 지정하지 않으면 순서대로 0,1,2... 값을 갖습니다.
따라서 Type.INACTIVE는 0 값을 갖습니다.

거꾸로 숫자로 Type형을 인식하게할 수도 있습니다. 예를 들어 Type[0] 은 Type.INACTIVE를 가리킵니다.


#### 2. 강제 지정

```ts
enum Type {
	INACTIVE,
	ACTIVE = 5,
	ALL
}
```

강제로 숫자를 지정하는 경우 그 다음부터는 ++ 되어 1씩 증가한 값을 갖습니다.
즉, Type.INACTIVE = 0, Type.ACTIVE = 5, Type.ALL = 6 이 됩니다.


##### shift를 활용한 강제 지정

```ts
enum Type {
	INACTIVE = 1 << 1,
	ACTIVE = 1 << 2,
	ALL = 1 << 3
}
```

숫자 강제 지정을 하다보면 순서가 꼬이거나 중복되는 경우가 생기므로 이를 간단하게 하기 위해 shift 연산을 사용합니다.


#### 3. Enum을 String으로

```ts
enum Type {
	INACTIVE,
	ACTIVE,
	ALL
}
```

Enum은 기본적으로 숫자 값을 갖는데 형태 그대로 String 값을 가져올 수도 있습니다.
Type.INACTIVE의 INACTIVE값을 0이 아닌 string으로 가져오려면 Type[Type.INACTIVE] 를 사용합니다.


#### 4. string 값 지정

```ts
enum Type {
	INACTIVE = 'inactive',
	ACTIVE = 'active',
	ALL = 'all'
}
```

string 값을 선언하면 숫자대신 정의된 string 값을 가져오게 됩니다.
즉, Type.INACTIVE 는 'inactive'가 됩니다.

반대로 'inactive'로 Type형을 가져오려면 'inactive' as Type 를 사용하면 됩니다.


##### 유의점
숫자형과 혼동해서 사용하지 않도록 모든 값을 선언해주는 것이 좋습니다.
문자형 뒤에 오는 enum 값은 선언하지 않으면 숫자형으로 인식하여 이전 값의 증가값을 찾는데 이전 값이 문자열이므로 증가할 수 없기에 에러를 발생하기 때문입니다.



#### 5. 함수값 지정

특이하게 Enum에 함수 값을 넣을 수 있습니다.

```ts
enum Type {
	INACTIVE = getInactiveValue(),
	ACTIVE = getActiveValue(),
	ALL // error
}
```

string값 지정할 때와 마찬가지로 모든 값은 선언되어야 합니다.


#### 6. Ambient Enum

```ts
enum Type {
	INACTIVE = 1,
	ACTIVE,
	ALL = 2
}
```

중복 값을 선언할 수 있습니다. 위와 같은 경우 Type.ACTIVE와 Type.ALL은 같은 값을 가집니다.
단, 역으로 Type[2] 값을 가져오려고 시도하는 경우 에러를 발생합니다.



#### 7. OR

```ts
enum Type {
	INACTIVE,
	ACTIVE,
	ALL
}

enum Active {
  type: Type.INACTIVE | Type.ACTIVE
}
```

위와 같이 OR를 사용한 값 설정도 가능합니다.


끝.



참고: https://www.typescriptlang.org/docs/handbook/enums.html