# NestJS에서 데코레이터 활용하기

## 데코레이터

☑️ Nest는 데코레이터를 적극 활용한다. 데코레이터를 잘 사용하면 횡단관심사를 분리하여 관점 지향 프로그래밍을 적용한 코드를 작성할 수 있다. 

☑️ 타입스크립트의 데코레이터는 파이썬의 데코레이터나 자바의 어노테이션과 유사한 기능을 한다.

☑️ **클래스, 메서드, 접근자, 프로퍼티, 매개변수**에 적용 가능하다. 각 요소의 선언부 앞에 `@`로 시작하는 데코레이터를 선언하면 데코레이터로 구현된 코드를 함께 실행한다.

☑️ 예를 들어 다음 코드는 유저 생성 요청의 본문을 DTO로 표현한 클래스이다.

```tsx
class CreateUserDto {
	@IsEmail()
	@MaxLength(60)
	readonly email: string;

	@IsString()
	@Matches(/^[A-Za-z\d!@#$%^&*()]{8,30}$/)
	readonly password: string;
}
```

☑️ 사용자는 얼마든지 요청을 잘못 보낼 수 있으므로 데코레이터를 이용하여 애플리케이션이 허용하는 값으로 제대로 요청을 보냈는지 검사하고 있다. 

- email은 이메일 형식을 가진 문자열이어야 하고 → `@IsEmail()`
- 그 길이는 최대 60자이어야 한다. → `@MaxLength(60)`
- password는 문자열이어야 하고 → `@IsString()`
- 주어진 정규 표현식에 적합해야 한다. → `@Matches(/^[A-Za-z\d!@#$%^&*()]{8,30}$/)`

<aside>
✅ 해당 기능은 tsconfig.json 파일에서 experimentalDecorators 옵션을 true로 설정해야만 사용할 수 있다.

</aside>

☑️ 데코레이터는 @expression 형식으로 사용된다. 여기서 expression은 데코레이팅 된 선언 (데코레이터가 선언되는 클래스, 메서드 등)에 대한 정보와 함께 런타임에 호출되는 함수여야 한다.

다음과 같은 메서드 데코레이터가 있고 이 데코레이터는 test라는 메서드에 선언했다. 여기서 deco 함수에 인자들이 있는데 메서드 데코레이터로 사용하기 위해서 이렇게 정의해야 한다.

```tsx
function deco(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
	console.log("데코레이터 평가됨");
}

class TestClass {
	@deco
	test() {
		console.log("함수 호출됨");
	}
}

const t = new TestClass();
t.test();
```

```
-- 콘솔 출력
데코레이터가 평가됨
함수 호출됨
```

## 데코레이터 함성

만약 여러개의 데코레이터를 사용한다면 수학에서의 함수 합성과 같이 적용된다.

다음 데코레이션 선언의 합성 결과는 $f(g(x))$와 같다

```tsx
@f
@g
test
```

여러 데코레이터를 사용할 때 다음 단계가 수행된다.

1. 각 데코레이터의 표현은 **위에서 아래로 평가(evaluate)**된다.
2. 그런 다음 결과는 **아래에서 위로 함수로 호출(call)**된다.

다음 예시의 출력 결과를 보면 합성 순서에 대한 이해를 높일 수 있다.

```tsx
function first() {
  console.log("first(): factory evalutaed");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("first(): called");
  };
}

function second() {
  console.log("second(): factory evaluated");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("second(): called");
  };
}

class ExampleClass {
  @first()
  @second()
  method() {
    console.log("method is called");
  }
}
```

```
--콘솔 출력
first(): factory evaluated
second(): factory evaluated
second(): called
first(): called
method is called
```

## 타입스크립트에서 지원하는 5가지 데코레이터

### 클래스 데코레이터 (Class Decorator)

☑️ 클래스 바로 앞에 선언된다.

☑️ 클래스 데코레이터는 클래스의 생성자에 적용되어 클래스 정의를 읽거나 수정할 수 있다. 

☑️ 선언 파일과 선언 클래스 내에서는 사용이 불가하다.

다음 코드는 클래스에 reportingURL 속성을 추가하는 클래스 데코레이터의 예이다.

```tsx
function reportableClassDecorator<T extends { new (...args: any[]): {} }>(constructor: T) { // 1
  return class extends constructor { // 2
    reportingURL = "http://www.example.com"; // 3
  };
}

@reportableClassDecorator
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}

const bug = new BugReport("Needs dark mode");
console.log(bug);
```

```
--콘솔출력
{type: 'report', title: 'Needs dark mode', reportingURL: 'http://www.example.com'}
```

- 클래스 데코레이터 팩토리이다. (1)
    - 생성자 타입(`new (…args: any[]): {}`. new 키워드와 함께 어떠한 형식의 인자들도 받을 수 있는 타입)을 상속받는 제네릭 타입 `T` 를 가지는 생성자(`constructor`)를 팩토리 메서드의 인자로 전달하고 있다.
- 클래스 데코레이터는 생성자를 리턴하는 함수여야 한다. (2)
- 클래스 데코레이터가 적용되는 클래스에 새로운 `reportingURL`이라는 속성을 추가한다. (3)
- BugReport 클래스의 선언되어 있지 않은 새로운 속성이 추가됐다.

<aside>
⚠️ 클래스의 타입이 변경되는 것은 아니다. 타입 시스템은 reportingURL을 인식하지 못하기 때문에 `bug.reportingURL`과 같이 직접 사용할 수 없다.

</aside>

### 메서드 데코레이터 (Method Decorator)

☑️ 메서드 바로 앞에 선언된다.

☑️ 메서드의 속성 디스크립터에 적용되고 메서드의 정의를 읽거나 수정할 수 있다.

☑️ 선언 파일, 오버로드 메서드, 선언 클래스에 사용이 불가하다.

메서드 데코레이터는 다음 세 가지의 인수를 가진다.

- **정적 멤버가 속한 클래스의 생성자 함수**이거나 **인스턴스 멤버에 대한 클래스의 프로토타입**
- **멤버의 이름**
- 멤버의 속성 디스크립터 → **PropertyDescriptor 타입을 가짐**

만약 메서드 데코레이터가 값을 반환한다면 이는 해당 메서드의 속성 디스크립터가 된다.

메서드 데코레이터의 예이다. 함수를 실행하는 과정에서 에러 발생시 이 에러를 잡아서 처리하는 로직이다.

```tsx
function HandleError() {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) { // 1
    console.log(target); // 2
    console.log(propertyKey); // 3
    console.log(descriptor); // 4

    const method = descriptor.value; // 5

    descriptor.value = function() { 
      try {
        method(); // 6
      } catch (e) {
        // error handling logic
        console.log(e);
      }
    }
  };
}

class Greeter {
  @HandleError()
  hello() {
    throw new Error("테스트 에러");
  }
}

const t = new Greeter();
t.hello();
```

- 메서드 데코레이터가 가져야 하는 3개의 인자이다. (1)
    - PropertyDescriptor는 객체 속성의 특성을 기술하고 있는 객체로, enumerable 외에도 여러가지 속성을 지닌다.
    - **enumerable이 true**면 이 속성을 **열거형**이라는 뜻이 된다.
    
    ```tsx
    interface PropertyDescriptor {
    	configurable?: boolean;
    	enumerable?: boolean;
    	value?: any;
    	writable?: boolean;
    	get?(): any;
    	set?(v: any): void;
    }
    ```
    
- target의 출력 결과는 `{constructor: f, greet: f}`이다. (2)
    - 데코레이터가 선언된 메서드 hello가 속해있는 클래스의 생성자와 프로토타입을 가지는 객체임을 알 수 있다.
- propertyKey의 출력 결과는 함수이름 `hello`이다. (3)
- hello 함수가 처음 가지고 있던 디스크립터가 출력된다. descriptor의 출력 결과는 `{value: f, writable: true, enumerable: false, configurable: true}` 이다. (4)
- 디스크립터의 value 속성으로 원래 정의된 메서드를 저장한다. (5)
- 원래의 메서드 호출 (6)

### 접근자 데코레이터 (Access Decorator)

☑️ 접근자 바로 앞에 선언한다.

☑️ 접근자의 속성 디스크립터에 적용되고 접근자의 정의를 읽거나 수정할 수 있다.

☑️ 선언 파일과 선언 클래스에 사용이 불가하다.

☑️ 접근자 데코레이터가 반환하는 값은 해당 멤버의 속성 디스크립터가 된다.

특정 멤버를 열거가 가능한지 결정하는 데코레이터의 예이다.

```tsx
function Enumerable(enumerable: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = enumerable; // 1
  }
}

class Person {
  constructor(private name: string) {} // 2

	// 3
  @Enumerable(true) 
  get getName() {
    return this.name;
  }

	// 4
  @Enumerable(false)
  set setName(name: string) {
    this.name = name;
  }
}

// 5
const person = new Person("Dexter");
for (let key in person) {
  console.log(`${key}: ${person[key]}`);
}
```

```
-- 콘솔 출력
name: Dexter
getName: Dexter
```

- 디스크립터의 enumerable 속성을 데코레이터의 인자로 결정한다. (1)
- name은 외부에서 접근하지 못하는 private 멤버이다. (2)
- 게터 `getName` 함수는 열거가 가능하도록 한다. (3)
- 세터 `setName` 함수는 열거가 불가능하도록 한다. (4)
- 출력 결과는 getName은 출력되지만, setName은 열거하지 못하기 때문에 for문에서 key로 받을 수 없다. (5)

### 속성 데코레이터 (Property Decorators)

☑️ 클래스의 속성 바로 앞에 선언된다.

☑️ 선언 파일, 선언 클래스에서 사용하지 못한다.

속성 데코레이터는 다음 두 가지의 인수를 가지는 함수이다.

- **정적 멤버가 속한 클래스의 생성자 함수**이거나 **인스턴스 멤버에 대한 클래스의 프로토타입**
- **멤버의 이름**

☑️ 메서드 데코레이터나 접근자 데코레이터와 비교했을 때 세 번째 인자인 속성 디스크립터가 존재하지 않는다.

☑️ 공식문서에 따르면 **반환값도 무시**되고, 이는 **현재 프로토타입의 멤버를 정의할 때 인스턴스 속성을 설명하는 메커니즘이 없고** **속성의 초기화 과정을 관찰하거나 수정할 수 있느 방법이 없기** 때문이라고 한다.

```tsx
function format(formatString: string) {
  return function (target: any, propertyKey: string): any {
    let value = target[propertyKey];

    function getter() {
      return `${formatString} ${value}`;
    }

    function setter(newVal: string) {
      value = newVal;
    }

    return {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true,
    }
  }
}

class Greeter {
  @format("Hello") // 1
  greeting: string;
}

const t = new Greeter();
t.greeting = "World";
console.log(t.greeting);
```

```
-- 콘솔 출력
Hello World
```

- 데코레이터에 formatString 전달 (1)

### 매개변수 데코레이터 (Parameter 데코레이터)

☑️ 생성자 또는 메서드의 파라미터에 선언되어 적용된다.

☑️ 선언 파일, 선언 클래스에 사용이 불가하다.

매개변수 데코레이터는 호출될 때 3가지의 인자와 함께 호출된다. (반환값은 무시된다.)

- **정적 멤버가 속한 클래스의 생성자 함수**이거나 **인스턴스 멤버에 대한 클래스의 프로토타입**
- **멤버의 이름**
- **매개변수가 함수에서 몇 번째  위치에 선언되었는지를 나타내는 인덱스**

<aside>
💡 NestJS에서 API 요청 파라미터에 대해 유효성 검사를 할 때 이와 유사한 데코레이터를 많이 사용한다.

</aside>

```tsx
import { BadRequestException } from "@nestjs/common";

function MinLength(min: number) { // 1
  return function (target: any, propertyKey: string, parameterIndex: number) {
    // 2
    target.validators = {
      minLength: function (args: string[]) { // 3
        return args[parameterIndex].length >= min; // 4
      }
    }
  }
}

function Validate(target: any, propertyKey: string, descriptor: PropertyDescriptor) { // 5
  const method = descriptor.value; // 6

  descriptor.value = function(...args) { // 7
		// 8
    Object.keys(target.validators).forEach(key => {
      if (!target.validators[key](args)) {
        throw new BadRequestException();
      }
    })
    method.apply(this, args); // 9
  }
}

class User {
  private name: string;

  @Validate
  setName(@MinLength(3) name: string) {
    this.name = name;
  }
}

const t = new User();
t.setName("Dexter"); // 10
console.log("------------");
t.setName("De"); // 11
```

- 파라미터의 최솟값을 검사하는 파라미터 데코레이터 (1)
- target 클래스 (여기서는 User)의 validators 속성에 유효성을 검사하는 함수를 할당한다. (2)
- args 인자는 (7)에서  넘겨받은 메서드의 인자이다. (3)
- 유효성 검사를 위한 로직이다. parameterIndex에 위치한 인자의 길이가 최솟값보다 같거나 큰지 검사한다. (4)
- 함께 사용할 메서드 데코레이터 (5)
- 메서드 데코레이터가 선언된 메서드를 method 변수에 임시 저장한다. (6)
- 디스크립터의 value에 유효성 검사 로직이 추가된 함수를 할당한다. (7)
- target(User 클래스)에 저장해둔 validators를 모두 수행한다. (8)
    - 이 때 원래 메서드에 전달된 인자(args)들을 각 validator에 전달한다.
- 원래 함수를 실행한다 (9)
- 파라미터 name의 길이가 5이므로 문제 없다. (10)
- 파라미터 name의 길이가 3보다 작으므로 BadRequestException이 발생한다. (11)

### 5가지 데코레이터 특징 정리

| 데코레이터 | 역할 | 호출시 전달되는 인자 | 선언 불가능한 위치 |
| --- | --- | --- | --- |
| 클래스 데코레이터 | 클래스의 정의를 읽거나 수정 | constructor | d.ts 파일, declare 클래스 |
| 메서드 데코레이터 | 메서드의 정의를 읽거나 수정 | target, propertyKey, propertyDescriptor | d.ts 파일, declare 클래스, 오버로드 메서드 |
| 접근자 데코레이터 | 접근자의 정의를 읽거나 수정 | target, propertyKey, propertyDescriptor | d.ts 파일, declare 클래스 |
| 속성 데코레이터 | 속성의 정의를 읽음 | target, propertyKey | d.ts 파일, declare 클래스 |
| 매개변수 데코레이터 | 매개변수의 정의를 읽음 | target, propertyKey, parameterIndex | d.ts 파일, declare 클래스 |