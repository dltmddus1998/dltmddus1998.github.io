# 객체

> 복잡한 객체 형태를 설명하는 방법과 타입스크립트가 객체의 할당 가능성을 확인하는 방법에 대해 다루어보자.
> 

## 1. 객체타입

{…} 구문을 사용해서 객체 리터럴을 생성하면, 타입스크립트는 해당 속성을 기반으로 새로운 객체 타입 또는 타입 형태를 고려한다. 해당 객체 타입은 객체의 값과 동일한 속성명과 원시 타입을 갖는다. 값의 속성에 접근하려면  value.멤버 또는 value[‘멤버’]구문을 사용한다.

```tsx
const poet = {
	born: 1935,
	name: 'Mary Oliver',
};

poet['born']; // 타입: number
poet.name; // 타입: string

poet.end; // Error: Property 'end' does not exist on type
```

객체 타입은 타입스크립트가 자바스크립트 코드를 이해하는 방법에 대한 핵심 개념이다.

null과 undefined를 제외한 모든 값은 그 값에 대한 실제 타입의 멤버 집합을 가지므로 타입스크립트는 모든 값의 타입을 확인하기 위해 객체 타입을 이해해야 한다.

### 1-1. 객체 타입 선언

객체의 타입을 명시적으로 선언하고 싶다

명시적으로 타입이 선언된 객체와는 별도로 객체의 형태를 설명하는 방법이 필요하다.

객체 타입은 객체 리터럴과 유사해보이지만 필드 값 대신 타입을 사용해 설명한다. 타입스크립트가 타입 할당 가능성에 대한 오류 메시지에 표시하는 것과 동일한 구문이다.

```tsx
let poetLater: {
	born: number;
	name: string;
}

// Ok
poetLater = {
	born: 1935,
	name: 'Mary Oliver',
};

poetLater = 'Sappho'; // Error: Type 'string' is not assignable to type
```

????

### 1-2. 별칭 객체 타입

위와 같이 객체 타입을 계속 작성하는 일은 매우 귀찮다.

각 객체 타입에 **타입 별칭을 할당**해 사용하는 방법이 더 일반적이다.

다음과 같이 이전 코드 스니펫은 Poet 타입으로 다시 작성할 수 있으며, 타입스크립트의 **할당 가능성 오류 메시지를 좀 더 직접적으로 읽기 쉽게 만드는 추가 이점**이 있다.

```tsx
type Poet = {
	born: number;
	name: string;
};

let poetLater: Poet;

// Ok
PoetLater = {
	born: 1935,
	name: 'Sara Teasdale',
};

poetLater = 'Emily Dickinson'; // Error: Type 'string' is not...
```

<aside>
📖 대부분의 타입스크립트 프로젝트는 객체 타입을 설명할 때 인터페이스(interface) 키워드를 사용하는 것을 선호한다. (7장 인터페이스 참고)
별칭 객체 타입과 인터페이스는 거의 동일하다.

</aside>

<aside>
💡 타입스크립트의 타입 시스템을 배울 때, **타입스크립트가 객체 리터럴을 해석하는 방법을 이해하는 것이 매우 중요**하다.

</aside>

## 2. 구조적 타이핑

타입스크립트의 타입 시스템은 **구조적으로 타입화(structurally typed)**되어 있다. 

매개변수나 변수가 특정 객체 타입으로 선언되면 타입스크립트에 어떤 객체를 사용하든 해당 속성이 있어야 한다.

```tsx
type WithFirstName = {
	firstName: string;
}

type WithLastName = {
	lastName: string;
}

const hasBoth = {
	firstName: 'Lucille',
	lastName: 'Clifton',
}

let withFirstName: WithFirstName = hasBoth;

let withLastName: WithLastName = hasBoth;
```

- Answer
    
    hasBoth 변수는 명시적으로 선언되지 않았음에도 두 개의 별칭 객체 타입을 모두 가지므로 두 개의 별칭 객체 타입 내에 선언된 변수를 모두 제공할 수 있다.
    

구조적 타이핑은 덕 타이핑(duck typing)과는 다르다.

~~덕타이핑은 ‘오리처럼 보이고 오리처럼 꽥꽥거리면, 오리일 것이다’라는 문구에서 유래했다.~~

✔️ 타입스크립트의 **타입 검사기에서 구조적 타이핑은 정적 시스템이 타입을 검사**하는 경우이다.

✔️ 덕 타이핑은 **런타임에서 사용될 때까지 객체 타입을 검사하지 않는 것**을 말한다.

**따라서 자바스크립트는 덕 타입, 타입스크립트는 구조적으로 타입화된다.**

### 2-1. 사용 검사

객체 타입으로 애너테이션된 위치에 값을 제공할 때 타입스크립트는 값을 해당 객체 타입에 할당할 수 있는지 확인한다. 할당하는 값에는 객체 타입의 필수 속성이 있어야 한다. 객체 타입에 필요한 멤버가 객체에 없다면 타입스크립트는 타입 오류를 발생시킨다.

```tsx
type FirstAndLastNames = {
	first: string;
	last: string;
}

// ???
const hasBoth: FirstAndLastNames = {
	first: 'Sarojini',
	last: 'Naidu',
}

// ???
const hasOnlyOne: FirstAndLastNames = {
	first: 'Sappho'
}
```

- Answer
    
    위 코드를 보면 두가지 속성을 모두 포함한 객체는 (hasBoth)는 문제 없지만 둘 중에 하나라도 포함하지 않는 객체는 오류를 발생시킨다.
    
    타입 별칭에서 지정하지 않은 타입도 허용하지 않는다. 예를들어 `first: 2000` 와 같이 생뚱맞은 타입의 값을 할당하면 이 또한 오류를 발생시킨다.
    

### 2-2. 초과 속성 검사

변수가 객체 타입으로 선언되고, 초깃값에 객체 타입에서 정의된 것보다 많은 필드가 있다면 타입스크립트에서 타입 오류가 발생한다. 따라서 **변수를 객체 타입으로 선언하는 것은 타입 검사기가 해당 타입에 예상되는 필드만 있는지 확인하는 방법**이기도 한다.

```tsx
type Poet = {
	born: number;
	name: string;
};

// Ok
const poetMatch: Poet = {
	born: 1928, 
	name: 'Maya Angelou'
};

// Type Error
const extraProperty: Poet = {
	activity: 'walking',
	born: 1928, 
	name: 'Maya Angelou'
};
```

**초과 속성 검사는 객체 타입으로 선언된 위치에서 생성되는 객체 리터럴에 대해서만 일어난다. (기존 객체 리터럴에 제공하면 초과 속성 검사를 우회한다.)**

<aside>
👉🏻 타입스크립트에서 초과속성을 금지하면 코드를 깨끗하게 유지할 수 있고, 예상한 대로 작동하도록 만들 수 있다.
**객체 타입이 선언되지 않은 초과 속성은 종종 잘못 입력된 속성 이름이거나 사용되지 않는 코드일 수 있다.**

</aside>

### 2-3. 중첩된 객체 타입

자바스크립트 객체는 다른 객체의 멤버로 중첩될 수 있으므로 타입스크립트의 객체 타입도 타입 시스템에서 중첩된 객체 타입을 나타낼 수 있어야  한다. 이를 구현하는 구문은 이전과 동일하지만 기본 이름 대신에 {…} 객체 타입을 사용한다.

```tsx
type Poem = {
	author: {
		firstName: string;
		lastName: string;
	};
	name: string;
};

// Ok
const poemMatch: Poem = {
	author: {
		firstName: 'Sylvia',
		lastName: 'Plath',
	},
	name: 'Lady Lazarus',
};

const poemMismatch: Poem = {
	author: {
		name: 'Sylvia Plath', // Error: Type '{name: string}' is not assignable
													// to type '{firstName: string, lastName: string}'...
	},
	name: 'Tulips'
};
```

Poem 타입을 작성할 때, author 속성의 형태를 자체 별칭 객체 타입으로 추출하는 방법도 있다. 중첩된 타입을 자체 타입 별칭으로 추출하면 타입스크립트의 타입 오류 메시지에 더 많은 정보를 담을 수 있다. 이 경우에는 { firstName: string, lastName: string; } 대신 Author를 사용할 수 있다.

```tsx
type Author = {
	firstName: string;
	lastName: string;
}

type Poem = {
	author: Author;
	name: string;	
}

const poemMismatch: Poem = {
	author: {
		name: "Sylvia Plath", // Error: Type '{name: string}' is not assignable
													// to type 'Author'
	}
}
```

✔︎ **위와 같이 코드를 수정하면 코드와 오류 메시지를 보다 쉽게 파악할 수 있다.**

### 2-4. 선택적 속성

모든 객체에 객체 타입 속성이 필요한건 아니다. 타입의 속성 애너테이션에서 : 앞에 ?를 추가하면 선택적 속성임을 나타낼 수 있다.

```tsx
type Book = {
	author: string | undefined;
	pages?: number;
}

const ok: Book = {
	author: undefined;
}

const error: Book = {
	// Error: Property author is missing in type '{}' but required in type 'Writers'
	pages: 20
}
```

위 코드를 보면 알 수 있듯, 선택적 속성과 `undefined`를 포함한 유니언 타입의 속성 사이에는 차이가 있다. ?를 사용해 선택적으로 선언된 속성은 존재하지 않아도 된다. 필수로 선언된 속성과 `| undefined`는 그 값이 `undefined`일지라도 반드시 존재해야 한다.

## 3. 객체 타입 유니언

> 타입스크립트 코드에서는 **속성이 조금 다른, 하나 이상의 서로 다른 객체 타입이 될 수 있는 타입**을 설명할 수 있어야 한다. 그리고 속성값을 기반으로 해당 객체 타입 간에 타입을 좁혀야 할 수도 있다.
> 

### 3-1. 유추된 객체 타입 유니언

변수에 여러 객체 타입 중 하나가 될 수 있는 **초깃값**이 주어지면 타입스크립트는 해당 타입을 객체 타입 유니언으로 유추한다. **유니언 타입**은 가능한 각 객체 타입을 구성하고 있는 요소를 모두 가질 수 있다. 

객체 타입에 정의된 각각의 가능한 속성은 비록 초깃값이 없는 선택적 (`?`) 타입이지만 각 객체 타입 구성 요소로 주어진다.

```tsx
const poem = Math.random() > 0.5 
	? { name: "The Double Image", pages: 7 }
	: { name: "Her Kind", rhymes: true};

poem.name; // string
poem.pages; // number | undefined
poem.rhymes; // booleans | undefined
```

위 코드에선 poem에 string값인 name은 항상 가지며 pages와 rhymes는 선택적이다.

### 3-2. 명시된 객체 타입 유니언

> 객체 타입의 조합을 명시하면 객체 타입을 더 명확히 정의할 수 있다.
> 

✔︎ 코드를 조금 더 작성해야 하지만 **객체 타입을 더 많이 제어할 수 있다**는 이점이 있다.

특히 값의 타입이 **객체 타입으로 구성된 유니언**이라면 **타입스크립트의 타입 시스템은 이런 모든 유니언 타입에 존재하는 속성에 대한 접근만 허용**한다.

```tsx
type PoemWithPages = {
	name: string;
	pages: number;
}

type PoemWithRhymes = {
	name: string;
	rhymes: boolean;
}

type Poem = PoemWithPages | PoemWithRhymes;

const peom: Poem = Math.random() > 0.5
	? { name: "The Double Image", pages: 7 },
	: { name: "Her Kind", rhymes: true };

poem.name; // OK

poem.pages; // Error: Property pages does not exist on type 'Poem'

peom.rhymes; // Error: Property rhymes does not exist on type 'poem'
```

위 코드에서 peom 변수에서 속성 name 같은 경우 항상 존재하므로 접근이 허용되지만 pages와 rhymes는 항상 존재한다는 보장이 없기 때문에 에러가 생기는 것이다. 

이와 같이 잠재적으로 존재하지 않는 객체의 멤버에 대한 접근을 제한하면 코드의 안전을 지킬 수 있다. ~~값이 여러 타입 중 하나일 경우, 모든 타입에 존재하지 않는 속성이 객체에 존재할 거라 보장할 수 없다.~~

💡 **리터럴 타입이나 원시 타입 모두, 혹은 둘 중 하나로 이루어진 유니언 타입에서 모든 타입에 존재하지 않은 속성에 접근하기 위해 타입을 좁혀야하는 것처럼 객체 타입 유니언도 타입을 좁혀야 한다.**

### 3-3. 객체 타입 내로잉

> **타입 검사기가 유니언 타입 값에 특정 속성이 포함된 경우에만 코드 영역을 실행할 수 있음을 알게 되면, 값의 타입을 해당 속성을 포함하는 구성 요소로만 좁힌다.**
> 

이를 **타입 내로잉**이라고 하는데, **코드에서 객체의 형태를 확인하고 타입 내로잉이 객체에 적용되는 것**이다.

💡 **poem의 pages가 타입스크립트의 타입 가드 역할을 해 PoemWithPages임을 나타내는지 확인한다. Poem이 PoemWithPages가 아니라면 PoemWithRhymes이어야 한다.**

```tsx
if ("pages" in poem) {
	poem.pages;
} else {
	poem.rhymes;
}
```

타입스크립트는 위 코드와 달리  `if (poem.pages)`와 같은 형식으로 참 여부를 확인하는 것을 **허용하지 않는다**. **존재하지 않는 객체의 속성에 접근하려고 시도하면 타입 가드처럼 작동하는 방식으로 사용되더라도 타입 오류로 간주된다.**

### 3-4. 판별된 유니언

> 자바스크립트와 타입스크립트에서 **유니언 타입으로 된 객체의 또 다른 인기 있는 형태는 객체의 속성이 객체의 형태를 나타내도록 하는 것**이다.
> 

이러한 타입 형태를 **판별된 유니언(discriminated union)**이라 하고 객체의 타입을 가리키는 속성을 **판별값**이라 한다.

```tsx
type PoemWithPages = {
	name: string;
	pages: number;
	type: 'pages';
};

type PoemWithRhymes = {
	name: string;
	rhymes: boolean;
	type: 'rhymes';
}

type Poem = PoemWithPages | PoemWithRhymes;

const peom: Poem = Math.random() > 0.5
	? { name: "The Double Image", pages: 7 },
	: { name: "Her Kind", rhymes: true };

if (poem.type === "pages") {
	console.log(`It's got pages: ${poem.pages}`); // OK
} else {
	console.log(`It rhymes: ${poem.rhymes}`);
}

poem.type; // 타입: 'pages' | 'rhymes'

poem.pages; // Error ...
```

위 코드에서 Poem 타입은 PoemWithPages 타입 또는 PoemWithRhymes 둘 다 될 수 있는 객체를 설명하고 type 속성으로 어느 타입인지를 나타낸다. 
만일 poem.type이 pages이면 타입스크립트는 poem을 PoemWithPages로 유추한다. **타입 내로잉 없이는 값에 존재하는 속성을 보장할 수 없다.**

이 코드는 자바스크립트 패턴과 타입스크립트의 타입 내로잉을 결합한 코드이다.

10장 ‘제네릭’과 관련된 프로젝트에서 제네릭 데이터 운영을 위해 판별된 유니언을 사용하는 방법을 자세히 살펴본다.

## 4. 교차 타입

> **타입스크립트 유니언 타입은 둘 이상의 다른 타입 중 하나의 타입이 될 수 있음을 나타낸다.**
> 

자바스크립트의 런타임 | 연산자가 & 연사자에 대응하는 역할을 하듯이, 타입스크립트에서도 **& 교차 타입**을 사용해 여러 타입을 동시에 나타낸다.

```tsx
type Artwork = {
	genre: string;
	name: string;
};

type Writing = {
	pages: number;
	name: string;
};

type WrittenArt = Artwork & Writing;
```

```tsx
type WrittenArt = {
	genre: string;
	name: string;
	pages: number;
}
```

위 두 코드의 WrittenArt는 같은 의미이다.

**교차 타입은 유니언 타입과 결합할 수 있으며, 이는 하나의 타입으로 판별된 유니언 타입을 설명하는 데 유용하다.**

```tsx
type ShortPoem = { author: string } & (
	| { kigo: string; type: "haiku"; }
	| { meter: number; type: "villanelle"; }
);

// Ok
const morningGlory: ShortPoem = {
	author: "Fukuda Chiyo-ni",
	kigo: "Morning Glory",
	type: "haiku",
};

const oneArt: ShortPoem = {
	// Error: Type: '{ author: string, type: "villanelle"; }'
	// is not assignable to type 'ShortPeom'
	auhor: "Elizabeth Biship",
	type: "villanelle"
}
```

ShortPoem 타입은 항상 author 속성을 가지며 하나의 type 속성으로 판별된 유니언 타입이다. 

### 4-1. 교차 타입의 위험성

교차 타입은 유용한 개념이지만, 타입스크립트 컴파일러를 혼동시키는 방식으로 사용하기 쉽다.

교차 타입을 사용할 때는 가능한 한 코드를 간결하게 유지해야 한다.

**긴 할당 가능성 오류** 

유니언 타입과 결합하는 것처럼 복잡한 교차 타입을 만들게 되면 할당 가능성 오류 메시지는 읽기 어려워진다. 즉, **복잡도가 높을수록 타입 검사기의 메시지도 이해하기 더 어려워진다.**

이 현상은 타입스크립트의 타입 시스템, 그리고 타입을 지정하는 프로그래밍 언어에서 공통적으로 관측된다.

위 코드 스니펫의 ShortPoem의 경우 **타입스크립트가 해당 이름을 출력하도록 타입을 일련의 별칭으로 된 객체 타입으로 분할하면 읽기가 훨씬 쉬워진다.** 

```tsx
type ShortPoemBase = { author: string };
type Haiku = ShortPoemBase & { kigo: string; type: 'haiku' };
type Villanelle = ShortPoemBase & { meter: number; type: 'villanelle' };
type ShortPoem = Haiku | Villanelle;

const oneArt: ShortPoem = {
	author: 'Elizabeth Bishop',
	type: 'villanelle'
};
```

**never**

교차 타입은 잘못 사용하기 쉽고 불가능한 타입을 생성한다. 원시 타입의 값은 동시에 여러 타입이 될 수 없기 때문에 교차 타입의 구성 요소로 함께 결합할 수 없다.

두개의 원시타입을 함께 시도하면 never 키워드로 표시되는 never 타입이 된다. 

```tsx
type NotPossible = number & string; // 타입: never
```

never 키워드와 never 타입은 프로그래밍 언어에서 bottom 타입 또는 empty 타입을 뜻합니다. bottom 타입은 값을 가질 수 없고 참조할 수 없는 타입이므로 bottom 타입에 그 어떠한 타입도 제공할 수 없다.

```tsx
let notNumber: NotPossible = 0; // Error

let notString: never = ""; // Error
```

대부분의 타입스크립트 프로젝트는 never 타입을 거의 사용하지 않지만 코드에서 불가능한 상태를 나타내기 위해 가끔 등장한다. 하지만 대부분 교차 타입을 잘못 사용해 발생한 실수일 가능성이 높다.
