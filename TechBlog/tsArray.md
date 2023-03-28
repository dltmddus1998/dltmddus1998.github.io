# 배열

## ✐ 자바스크립트에서의 배열

자바스크립트 배열은 매우 유연하고 내부에 모든 타입의 값을 혼합해서 저장할 수 있다.

대부분의 개별 자바스크립트 배열은 하나의 특정 타입의 값만 가진다.

→ **다른 타입의 값을 추가하게 되면 배열을 읽을 때 혼란을 줄 수 있으며**, 최악의 상황으로는 프로그램에 문제가 될 만한 오류가 발생할 수도 있다.

## ✐ 타입스크립트에서의 배열

타입스크립트는 **초기 배열에 어떤 데이터 타입이 있는지 기억**하고, **배열이 해당 데이터 타입에서만 작동하도록 제한**한다. 이런 방식으로 **배열의 데이터 타입을 하나로 유지**시킨다.

## 1. 배열 타입

> 배열을 저장하기 위한 변수는 초깃값이 필요하지 않다.
> 

→ 변수는 undefined로 시작해서 나중에 배열 값을 받을 수 있다. 

타입스크립트는 **변수에 타입 애너테이션을 제공해 배열이 포함해야 하는 값의 타입을 알려주려고 한다**. 

배열에 대한 타입 애너테이션은 배열의 요소 타입 다음에 []가 와야 한다.

```tsx
let arrayOfNumbers: number[];

arrayOfNumbers = [4, 8, 15, 16, 23, 42];
```

### 1-1. 배열과 함수 타입

> 배열 타입은 함수 타입에 무엇이 있는지를 구별하는 괄호가 필요한 구문 컨테이너의 예이다.
> 

→ 괄호는 애너테이션의 어느 부분이 함수 반환 부분이고 어느 부분이 배열 타입 묶음인지를 나타내기 위해 사용한다. 

```tsx
// 타입은 string 배열을 반환하는 함수
let createStrings: () => string[];

// 타입은 각각의 string을 반환하는 함수 배열
let stringCreators: (() => string)[];
```

💡 **위 각각의 타입은 반환하는 값이 다름을 알 수 있다.**

### 1-2. 유니언 타입 배열

> **배열의 각 요소가 여러 선택 타입 중 하나일 수 있음을 나타내려면 유니언 타입을 사용**한다.
> 

유니언 타입으로 배열 타입을 사용할 때 **애너테이션의 어느 부분이 배열의 콘텐츠이고 어느 부분이 유니언 타입 묶음인지를 나타내기** 위해 **괄호를 사용**해야 할 수도 있다.

👉🏻 **아래 코드의 두 타입은 동일하지 않다.**

```tsx
// 타입은 string 또는 number의 배열
let stringOrArrayOfNumbers: string | number[];

// 타입은 각각 number 또는 string인 요소의 배열
let arrayOfStringOrNumbers: (string | number)[];
```

### 1-3. any 배열의 진화

> 초기에 빈 배열로 설정된 변수에서 타입 애너테이션을 포함하지 않으면 타입스크립트는 배열을 any[]로 취급하고 모든 콘텐츠를 받을 수 있다.
> 

→ **그러나**, **타입 애너테이션이 없는 빈 배열은 잠재적으로 잘못된 값 추가를 허용해 타입스크립트의 타입 검사기가 갖는 이점을 부분적으로 무력화**하므로 이는 선호하지 않는다.

```tsx
// 타입: any[]
let values = [];

// 타입: string[]
values.push('');

// 타입: (number | string)[]
values[0] = 0;
```

<aside>
변수와 마찬가지로 배열이 any 타입이 되도록 허용하거나 일반적으로 any 타입을 사용하도록 허용하면 타입스크립트의 타입 검사 목적을 부분적으로 무효화한다.

</aside>

### 1-4. 다차원 배열

> 2차원 배열 또는 배열의 배열은 두 개의 []를 갖는다.
> 

```tsx
// 타입: number[][]
let arrayOfSArraysOfNumbers: (number[])[];
```

## 2. 배열 멤버

<aside>
타입스크립트는 배열의 멤버를 찾아서 해당 배열의 타입 요소를 되돌려주는 전형적인 인덱스 기반 접근 방식을 이해하는 언어이다.

</aside>

유니언  타입으로 된 배열의 멤버는 그 자체로 동일한 유니언 타입이다.

```tsx
const soldiersOrDates = ["Deborah Sampson", new Date(1782, 6, 3)];

// 타입: string || Date - 해당 변수는 string | Date 타입이다.
const soldierOrDate = soldiersOrDates[0];
```

### 2-1. 주의 사항: 불안정한 멤버

> 타입스크립트 타입 시스템은 기술적으로 불안정하다고 알려져 있다.
> 

→ 대부분 올바른 타입을 얻을 수 있지만, 때로는 값 타입에 대한 타입 시스템의 이해가 올바르지 않을 수 있다.

특히 배열은 타입 시스템에서 불안정한 소스이다.

기본적으로 타입스크립트는 모든 배열의 멤버에 대한 접근이 해당 배열의 멤버를 반환한다고 가정하지만, 자바스크립트에서조차도 **배열의 길이보다 큰 인덱스로 배열 요소에 접근하면 undefined를 제공**한다. 

```tsx
function withElements(elements: string[]) {
	console.log(elements[9001].length); // 타입 오류 없음
}

withElements(["It's", "over"]);
```

타입스크립트는 검색된 배열의 멤버가 존재하는지 의도적으로 확인하지 않아서 elements[9001]은 undefined가 아니라 string 타입으로 간주된다.

<aside>
타입스크립트에는 배열 조회를 더 제한하고 타입을 안전하게 만드는 noUncheckedIndexedAccess 플래그가 있지만 이 플래그는 매우 엄격해서 대부분의 프로젝트에서 사용하지 않는다.

</aside>

## 3. 스프레드와 나머지 매개변수

**… 연산자를 사용하는 나머지 매개변수**와 **배열 스프레드**는 자바스크립트에서 배열과 상호작용하는 핵심 방법이다.

타입스크립트는 두 방법을 모두 이해한다.

### 3-1. 스프레드

… 스프레드 연산자를 사용해 배열을 결합한다. 

```tsx
// 타입: string[]
const soldiers = ["Harriet Tubman", "Joan of Arc", "Khutulun"];

// 타입: number[]
const soldierAges = [90, 19, 45];

// 타입: (string | number)[]
const conjoined = [...soldiers, ...soldierAges];
```

conjoined 배열은 string 타입과 number 타입 값을 모두 포함하므로 (string | number)[] 타입으로 유추된다.

### 3-2. 나머지 매개변수 스프레드

```tsx
function logWarriors(greeting: string, ...names: string[]) {
	for (const name of names) {
		console.log(`${greeting}, ${name}!`);
	}
}

const warriors = ["Cathy Williams", "Lozen", "Nzinga"];

logWarriros("Hello", ...warriors);

const birthYears = [1844, 1840, 1583];

logWarriors("Born in", ...birthYears); // Error:Argument of type number is not assignable to parameter of type 'string'
```

logWarriors는 두 번째 인자에 string으로 구성된 배열을 스프레드한 값만 허용되므로 에러가 났다.

타입스크립트는 나머지 매개변수로 배열을 스프레드하는 자바스크립트 실행을 인식하고 이에 대해 타입 검사를 수행한다.

→ 나머지 매개변수를 위한 인수로 사용되는 배열은 나머지 매개변수와 동일한 배열 타입을 가져야 한다.

## 4. 튜플

<aside>
자바스크립트 배열은 이론상 어떤 크기라도 될 수 있지만 때로는 **튜플**이라고 하는 **고정된 크기의 배열**을 사용하는 것이 유용하다.

</aside>

> 튜플 배열은 각 인덱스에 알려진 특정 타입을 가지며 배열의 모든 가능한 멤버를 갖는 유니언 타입보다 더 구체적이다.
> 

> 튜플 타입을 선언하는 구문은 배열 리터럴처럼 보이지만 요소의 값 대신 **타입**을 적는다.
> 

```tsx
let yearAndWarrior: [number, string];

yearAndWarrior = [530, "abc"]; // Ok

yearAndWarrior = [false, "cde"]; // Error: Type boolean is not assignable to type 'number'
```

위 배열은 인덱스 0에 number 타입을 가져야 하는데 false라는 boolean 타입이 들어가 있으므로 에러가 난 것이다.

<aside>
자바스크립트에서는 단일 조건을 기반으로 두 개의 변수에 초깃값을 설정하는 것처럼 한 번에 여러 값을 할당하기 위해 **튜플**과 **배열** **구조 분해 할당**을 함께 자주 사용한다.

</aside>

```tsx
// year type: number
// warrior type: string

let [year, warrior] = Math.random() > 0.5
	? [340, 'abc']
	: [1828, 'cde'];
```

### 4-1. 튜플 할당 가능성

<aside>
타입스크립트에서 튜플 타입은 가변 길이의 배열 타입보다 더 구체적으로 처리된다.

</aside>

→ 가변 길이의 배열 타입은 튜플 타입에 할당될 수 없다.

```tsx
// type: (boolean | number)[]
const pairLoose = [false, 123];
const pairTupleLoose: [boolean, number] = pairLoose;
```

pairLoose가 [boolean, number] 자체로 선언된 경우 pairTupleLoose에 대한 값 할당이 허용되겠지만 타입스크립트는 튜플 타입의 튜플에 얼마나 많은 멤버가 있는지 알고 있으므로 길이가 다른 튜플은 서로 할당할 수 없다.

🧐 **나머지 매개변수로서의 튜플**

<aside>
튜플은 구체적인 길이와 요소 타입 정보를 가지는 배열로 간주되므로 함수에 전달할 인수를 저장하는 데 특히 유용하다.

</aside>

→ 타입스크립트는 … 나머지 매개변수로 전달된 튜플에 정확한 타입 검사를 제공할 수 있다.

```tsx
function logPair(name: string, value: number) {
	console.log(`${name} has ${value}`);
}

const pairArray = ['Amage', 1];
logPair(...pairArray); // Error

const pairTupleIncorrect: [number, string] = [1, 'Amage'];
logPair(...pairTupleIncorrect); // Error

const pairTupleCorrect: [string, number] = ['Amage', 1];
logPair(...pairTupleCorrect); // Ok
```

위 코드와 같이 값이 [string, number] 튜플이라고 알고 있다면 값이 일치하다는 것을 알게 된다.

### 4-2. 튜플 추론

<aside>
타입스크립트는 **생성된 배열을 튜플이 아닌 가변 길이의 배열로 취급**한다.

</aside>

→ 배열이 변수의 초깃값 또는 함수에 대한 반환값으로 사용되는 경우, 고정된 크기의 튜플이 아니라 **유연한 크기의 배열로 가정**한다.

타입스크립트에서는 값이 일반적인 배열 타입 대신 **좀 더 구체적인 튜플 타입이어야 함**을 다음 두 가지 방법으로 나타낸다. 

✔︎ **명시적 튜플 타입**

> 함수에 대한 반환 타입 애너테이션처럼 튜플 타입도 타입 애너테이션에 사용 가능하다.
> 

→ 함수가 **튜플 타입을 반환한다고 선언**되고, **배열 리터럴을 반환한다**면 해당 배열 리터럴은 일반적인 가변 길이의 배열 대신 **튜플로 간주**된다.

```tsx
// 반환 타입: [string, number]
function firstCharAndSizeExplicit(input: string): [string, number] {
	return [input[0], input.length];
}

const [firstChar, size] = firstCharAndSizeExplicit("Cathay Williams");
```

✔︎ **const 어서션**

> 명시적 타입 애너테이션에 튜플 타입을 입력하는 작업은 명시적 타입 애너테이션을 입력할 때와 동일한 이유로 고통스러울 수 있다.
> 

→ 코드 변경에 따라 작성 및 수정이 필요한 구문을 추가해야 한다.

😉 대안으로 타입스크립트는 값 뒤에 넣을 수 있는 const 어서션인 as const 연산자를 제공한다.

> **const 어서션**은 타입스크립트에 **타입을 유추할 때 읽기 전용이 가능한 값 형식을 사용하도록** 지시한다.
> 

```tsx
const unionArray = [1111, 'abc'];

const readonlyTuple = [1111, 'abc'] as const;
```

💡 **위와 같이 배열 리터럴 뒤에 as const가 배치되면 배열이 튜플로 처리되어야 한다.**

<aside>
**const 어서션은 유연한 크기의 배열을 고정된 크기의 튜플로 전환하는 것을 넘어서, 해당 튜플이 읽기 전용이고 값 수정이 예상되는 곳에서 사용할 수 없음을 나타낸다.**

</aside>
