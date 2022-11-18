# 객체지향 프로그래밍

## OOP (Object Oriented Programming)

> **a programming paradigm based on the concept of objects - 객체를 개념으로 한 프로그래밍 방식
Object(객체): 관련된 데이터나 코드를 함께 묶는 것**
> 

# 객체지향 개념 정리 (↔ 절차적 프로그래밍)

## Imperative and Procedural Programming

> **명령과 절차를 따르는 프로그래밍
하나의 어플리케이션을 만들 때 어플리케이션이 동작하기 위한 데이터와 함수들 위주로 구성하는 것
정의된 순서대로 함수를 하나씩 호출하는 방식**
> 

**절차적 프로그래밍의 치명적인 단점**

- 함수가 여러 가지가 얽혀있고 데이터도 다르곳에서 업데이트될 수 있으므로 하나를 수정하기 위해 전체적인 어플리케이션이 어떻게 동작하는지 이해해야 한다.
    - 하나를 수정했을 때 다른 side effect가 발생할 확률이 높다.
- 한눈에 어플리케이션을 이해하기 어렵다.
- 유지보수 및 확장이 어렵다.

**객체지향 프로그래밍 장점**

```
**- 프로그램을 객체로 정의해서 객체들끼리 서로 의사소통하도록 디자인하고 코딩한다.
- 서로 관련있는 데이터와 함수를 여러가지 오브젝트로 정의해서 프로그래밍한다.
- 객체지향이라는 것은 “우리가 흔히 볼 수 있는 물건들 오브젝트 객체들”을 말한다.
- 한 곳에서 문제가 생긴다면 관련있는 오브젝트만 이해하고 수정하면 된다.
- 여러 번 반복되는 것이 있다면 관련 오브젝트 재사용이 가능하다.
- 새로운 기능 필요시 새로운 오브젝트를 만들면 되므로 확장성이 올라간다.
- 생산성 up
- higher-quality
- 속도 up
- 유지보수 용이**
```

**Object 구성**

데이터(data, fields, properties) + 함수(methods)

Object는 우리 주변에서 볼 수 있는 학생, 은행 등 다양한 객체들을 선정해서 디자인해볼 수 있다. 이런 일상에서 볼 수 있는 물체뿐 아니라 프로그래밍할 때 만날 수 있는 추상적인 컨셉인 error, exception, event 등도 object로 만들고 관리할 수 있다. 

## class & object

> **class: 단순 틀 같은 개념 (데이터가 들어있지 않다.)
어떻게 생겼는지 묘사하는 역할
- template
- declare once
- no data in**
> 

> **object: 클래스에 데이터를 넣어 만듦
- instance of a class
- created many times
- data in**
> 

object는 class의 instance라고 얘기한다. 

~~ex) 붕어빵 클래스를 이용해 팥 붕어빵 인스턴스를 생성했다.~~

<aside>
💡 클래스라는 청사진을 이용해서 다양한 데이터를 넣어 많은 오브젝트를 만들 수 있다.

</aside>

## 🎉 중요한 객체지향 원칙


### 캡슐화 (Encapsulation)

> **절차지향 프로그래밍에서는 데이터와 함수 등 여러가지가 섞여 있다. 
이렇게 흩어져 있는 것들 중 서로 관련있는 것들을 묶어놓는 것을 캡슐화라 한다.
즉, 서로 관련 있는 데이터와 함수를 한 오브젝트 안에 담아두고 외부에서 보일 필요가 없는 데이터를 잘 숨겨서 캡슐화를 한다.**
> 

### 추상화 (Abstraction)

> **외부의 복잡함을 모두 이해하지 않고 외부에서 간단한 인터페이스를 통해 쓸 수 있는 것을 말한다.
예를 들어, 커피 머신을 이용할 때 내부의 복잡한 매커니즘을 모르고도 외부의 버튼만을 이용해 사용할 수 있다.
이처럼 추상화를 통해 외부에서는 내부가 얼마나 복잡한지 상관없이 지정된 외부에서만 보이는 인터페이스 함수를 이용해서 오브젝트를 사용할 수 있다.**
> 

### 상속성 (Inheritance)

> **상속을 이용하면 한번 잘 정의해둔 클래스를 재사용할 수 있다.**
> 

상속의 관계: 부모 클래스 ↔ 자식 클래스 (IS-A 관계 → 상속을 받은 자식 클래스는 바로 부모 클래스이다.)

<aside>
💡 브라우저에서 사용되는 DOM 요소도 모두 상속을 통해 구현된다.
HTMLElement라는 클래스는 Element라는 클래스를 상속받았다. 즉, HTMLElement는 Element라고 말할 수 있다. 
Element는 Node를 상속받았고 Node는 EventTarget을 상속받았다. 
Element는 EventTarget뿐만 아니라 Node에 있는 모든 것들을 이용할 수 있다.

</aside>

### 다형성 (Polymorphism)

> **상속을 통해 만들어진 객체들이 무엇인지와 상관없이 공통된 함수를 호출할 수 있다.**
> 

## ？객체지향적으로 커피기계 만들기

### 절차지향적으로 커피기계 만들기

```tsx
{
    type CoffeeCup = {
        shots: number;
        hasMilk: boolean;
    }

    const BEANS_GRAMM_PER_SHOT: number = 7;
    let coffeeBeans: number = 0;
    function makeCoffee(shots: number): CoffeeCup {
        if (coffeeBeans < shots * BEANS_GRAMM_PER_SHOT) {
            throw new Error('Not enough coffee beans!!');
        }
        coffeeBeans -= shots * BEANS_GRAMM_PER_SHOT;
        return {
            shots,
            hasMilk: false
        };
    }

    coffeeBeans += 3 * BEANS_GRAMM_PER_SHOT;
    const coffee = makeCoffee(2);
    console.log(coffee);
    
}

// 출력
// { shots: 2, hasMilk: false }
```

### 1️⃣ 객체지향적으로 커피기계 만들기 (static 사용)

```tsx
{
    type CoffeeCup = {
        shots: number;
        hasMilk: boolean;
    }

    class CoffeeMaker {
        static BEANS_GRAMM_PER_SHOT: number = 7;
        coffeeBeans: number = 0;

        constructor(coffeeBenas: number) {
            this.coffeeBeans = coffeeBenas; // 해당 클래스 안에 있는 coffeeBeans를 전달된 coffeeBeans만큼 할당
        }

        makeCoffee(shots: number): CoffeeCup {
            if (this.coffeeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
                throw new Error('Not enough coffee beans!!');
            }
            this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
            return {
                shots,
                hasMilk: false
            };
        }
    }

    const maker = new CoffeeMaker(32);
    console.log(maker);

    const maker2 = new CoffeeMaker(34);
    console.log(maker2);
    
    
}

// 출력
// CoffeeMaker { BEANS_GRAMM_PER_SHOT: 7, coffeeBeans: 32 }
```

<aside>
💡 내 클래스 안에 있는 멤버변수에 접근할 때는 this를 사용해야 한다.

</aside>

<aside>
💡 constructor : 클래스로 오브젝트 인스턴스를 만들 때 항상 호출되는 함수

</aside>

<aside>
💡 `BEANS_GRAMM_PER_SHOT`의 경우 클래스 내부에서 연결된 정보이고 변하지 않는 상수이다. 그런데 이렇게 멤버변수로 작성하게 되면 클래스로 만드는 오브젝트마다 해당 상수 (7)이 들어있게된다. 
이처럼 클래스에서 한번 정의되고 클래스를 이용한 오브젝트 사이에서 모두 공유될 수 있는 것들은 멤버변수로 두면 오브젝트 생성할 때마다 중복되어 나타나므로 메모리 낭비 가능성이 높다. 
해결책으로는 `static`이라는 키워드를 붙이면 class level로 분류된다.
class level은 class와 연결되므로 오브젝트 생성할 때마다 나타나지 않는다.여기서 class level은 클래스 안에 있는게 아닌 클래스 자체에 있는 것이므로 this를 사용하지 않는다.
대신, 클래스 이름을 지정해야 한다. (`this` → `CoffeeMaker`)
(붙이지 않은 것은 instance|object level이라 불린다.)

</aside>

<aside>
💡 클래스는 관련된 속성과 함수들을 묶어서 어떤 모양의 데이터가 될거라는 걸 정의하는 것이고, 이 클래스에 실제 데이터를 넣어서 오브젝트를 만들 수 있다.

</aside>

### 만약에 constructor을 사용하고 싶지 않다?

```tsx
{
    type CoffeeCup = {
        shots: number;
        hasMilk: boolean;
    }

    class CoffeeMaker {
        static BEANS_GRAMM_PER_SHOT: number = 7;
        coffeeBeans: number = 0;

        constructor(coffeeBenas: number) {
            this.coffeeBeans = coffeeBenas; // 해당 클래스 안에 있는 coffeeBeans를 전달된 coffeeBeans만큼 할당
        }

        static makeMachine(coffeeBeans: number): CoffeeMaker {
            return new CoffeeMaker(coffeeBeans);
        }

        makeCoffee(shots: number): CoffeeCup {
            if (this.coffeeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
                throw new Error('Not enough coffee beans!!');
            }
            this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
            return {
                shots,
                hasMilk: false
            };
        }
    }

    const maker = new CoffeeMaker(32);
    console.log(maker);

    const maker2 = new CoffeeMaker(34);
    console.log(maker2);
    
    const maker3 = CoffeeMaker.makeMachine(3);
}
```

<aside>
💡 클래스 내부의 어떠한 속성값도 필요로 하지 않으므로 앞에 static을 붙여주면 된다.
이를 통해, 외부에서도 클래스 만들지않고 `CoffeeMaker` 클래스에 있는 `makeMachine` 함수를 통해 간단하게 커피기계를 만들 수 있다.
만약 static을 붙이지 않는다면, `CoffeeMaker.makeMachine()`은 사용불가하고, 위에 만들어진 오브젝트를 통해 (`maker.makeMachine()`) 함수를 호출할 수 있다.

</aside>

### 2️⃣ 캡슐화 (encapsulation) 적용하기

> **캡슐화는 클래스를 만들 때 외부에서 접근할 수 있는 것은 무엇인지 내부적으로만 가져야 할 데이터는 무엇인지와 같은 것들을 결정할 수 있다.**
> 

<aside>
💡 위 코드의 문제점!
`maker.coffeBeans = -4;` 이런식으로 비정상적인 오브젝트의 변수 할당에 대한 제약사항이 없으므로 외부에서 오브젝트 상태를 유효하지 않은 상태로 만들 수 있다.
해당 문제를 캡슐화를 이용하여 외부에서 보이면 안되는 부분을 가릴 것이다.

</aside>

```tsx
{
    type CoffeeCup = {
        shots: number;
        hasMilk: boolean;
    }

    class CoffeeMaker {
        private static BEANS_GRAMM_PER_SHOT: number = 7;
        private coffeeBeans: number = 0;

        private constructor(coffeeBenas: number) {
            this.coffeeBeans = coffeeBenas; // 해당 클래스 안에 있는 coffeeBeans를 전달된 coffeeBeans만큼 할당
        }   

        static makeMachine(coffeeBeans: number): CoffeeMaker {
            return new CoffeeMaker(coffeeBeans);
        }

        fillCoffeeBeans(beans: number) {
            if (beans < 0) {
                throw new Error('values for beans should be greater than 0');
            }
            this.coffeeBeans += beans;
        }

        makeCoffee(shots: number): CoffeeCup {
            if (this.coffeeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
                throw new Error('Not enough coffee beans!!');
            }
            this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
            return {
                shots,
                hasMilk: false
            };
        }
    }

    const maker = CoffeeMaker.makeMachine(32);
    maker.fillCoffeeBeans(32);
    console.log(maker);
}
```

<aside>
💡 `public`, `private`, `protected`를 이용하여 다양한 레벨의 정보를 은닉할 수 있다.
따로 작성하지 않으면 public으로 간주한다.

</aside>

<aside>
💡 외부에서는 `BEANS_GRAMM_PER_SHOT`은 보일 필요가 없으므로 `private`으로 지정하면 된다.
`coffeeBeans`의 경우에는, 내부의 상태는 `private`으로 숨겨놓고 외부에서는 `fillCoffeeBeans`라는 함수를 이용해서 커피콩을 채울 수 있게 한다.
이런 함수를 이용함으로써, 전달받은 인자의 유효성을 검사해서 높은 안정성의 코딩이 가능해진다.

</aside>

<aside>
💡 `private`은 외부에서는 절대 볼 수도 없고 접근할 수도 없는 상태로 만든다. 
`protected`는 상속시 외부에서는 접근할 수 없지만, 클래스를 상속한 자식 클래스에서만 접근이 가능하도록 설정할 수 있다.

</aside>

<aside>
💡 `static`을 붙여서 무언가 오브젝트를 만들 수 있는 함수를 제공한다는 것은 즉 누군가 이런 생성자를 이용해서 생성하는 것을 금지하기 위해 사용한다.
이럴 경우, `constructor`을 `private`으로 만들어서 항상 `static` 메소드를 이용할 수 있도록 권장하는 것이 좋다.

</aside>

### 3️⃣ 유용한 Getter와 Setter

> setter와 getter은 일반 멤버변수처럼 사용 가능한데, 어떠한 계산시 좀 더 유용하게 사용할 수 있다.
> 

```tsx
class User {
    get fullName(): string { 
        return `${this.firstName} ${this.lastName}`;
    }
    private internalAge = 4;
    get age(): number {
        return this.internalAge;
    }
    set age(num: number) {
        if (num < 0) {
            throw new Error('values for age should be greater than 0')
        }
        this.internalAge = num;
    }
    constructor(private firstName: string, private lastName: string) {

    }
}
const user = new User('Steve', 'Jobs');
user.age = 6; // 내부적으로 internalAge 접근은 불가하지만 set age()를 통해 internalAge 값 할당 가능
console.log(user.fullName); // Ellie Jobs
console.log(user.age);

// 출력
Steve Jobs
6
```

<aside>
💡 `fullName()`이 함수 형태이지만 접근할 때는 멤버변수에 접근하는 형태로 사용하면 된다.

`set age()`를 통해 `private`한 `internalAge`의 값에 할당이 가능해진다.

전달된 숫자가 정확한지에 대해 유효성 검사를 할 수 있다.
이와 같이, `getter`와 `setter`을 통해 좀 더 다양한 연산이 가능하다.

</aside>

<aside>
💡 `constructor`의 인자 `firstName`, `lastName`은 다음과 같이 `private`을 앞에 붙여주면 아래 코드와 같은 의미이지만 훨씬 코드가 간결해진다.

</aside>

```tsx
private firstName: string;
private lastName: string;

.
.
.
constructor(firstName: string, lastName: string) {
	this.firstName = firstName;
	this.lastName = lastName;
}
```

### 4️⃣ 추상화 (Abstraction)

```tsx
{
    type CoffeeCup = {
        shots: number;
        hasMilk: boolean;
    }

    // 2. 인터페이스 정의를 통한 추상화
    // CoffeeMaker라는 인터페이스를 이용하면 makeCoffee를 이용할 수 있다는 것 명시
    interface CoffeeMaker {
        makeCoffee(shots: number): CoffeeCup;
    }

    class CoffeeMachine implements CoffeeMaker {
        private static BEANS_GRAMM_PER_SHOT: number = 7;
        private coffeeBeans: number = 0;

        private constructor(coffeeBenas: number) {
            this.coffeeBeans = coffeeBenas; // 해당 클래스 안에 있는 coffeeBeans를 전달된 coffeeBeans만큼 할당
        }   

        static makeMachine(coffeeBeans: number): CoffeeMachine {
            return new CoffeeMachine(coffeeBeans);
        }

        fillCoffeeBeans(beans: number) {
            if (beans < 0) {
                throw new Error('values for beans should be greater than 0');
            }
            this.coffeeBeans += beans;
        }

        private grindBeans(shots: number) {
            console.log(`grinding beans for ${shots}`);
            if (this.coffeeBeans < shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT) {
                throw new Error('Not enough coffee beans!!');
            }
            this.coffeeBeans -= shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT;
        }

        private preheat(): void {
            console.log('heating up...🔥');
            
        }

        private extract(shots: number): CoffeeCup {
            console.log(`Pulling ${shots} shots...☕️`);
            return {
                shots,
                hasMilk: false,
            }
            
        }

        makeCoffee(shots: number): CoffeeCup {
            this.grindBeans(shots);
            this.preheat();
            return this.extract(shots);
        }
    }

    const maker: CoffeeMachine = CoffeeMachine.makeMachine(32);
    maker.fillCoffeeBeans(32);
    maker.makeCoffee(2);
}
```

> **grindBeans, preheat, extract 이 세 함수들을 추가했을 때, 어떤 순서대로 써야할지 모르겠다... 
이 때 추상화가 빛을 발하게 된다.**
> 

> **추상화는 인터페이스를 간단하게 만듦으로써 사용자가 간편하게 사용할 수 있도록 도와준다.**
> 

> **보통 정보은닉 (캡슐화, encapsulation)을 통해 추상화를 할 수 있다.**
> 

<aside>
💡 첫번째는, 위 코드 처럼 필요한 함수만 노출해서 (그 외는 private을 통해 캡슐화 진행) 양식을 간단하게 만든 방식이다.

</aside>

<aside>
💡 두번째는, 인터페이스를 이용해서 만드는 방식이다.
→ 인터페이스는 어떤 행동을 할건지 명시해 놓는 계약서라고 생각하면 된다.

</aside>

> **인터페이스는 외부에서 사용하므로 최대한 `CoffeeMaker`라는 이름을 유지하고 구현하는 클래스에서 다른 이름을 가져가는 것을 지향한다.
따라서 해당 클래스는 `CoffeeMachine`으로 변경한다.**
> 

> **이 클래스는 위 인터페이스 규격을 따른다. (`implements CoffeeMaker`)
따라서 해당 클래스는 해당 인터페이스에서 규격된 모든 함수를 구현해야 한다. 해당 클래스에서 `makeCoffee`를 구현하지 않으면, 클래스 자체에 에러가 발생한다.**
> 

### +) 전문샵에서 사용하는 CoffeeMaker Interface

```tsx
{
    type CoffeeCup = {
        shots: number;
        hasMilk: boolean;
    }

    // 2. 인터페이스 정의를 통한 추상화
    // CoffeeMaker라는 인터페이스를 이용하면 makeCoffee를 이용할 수 있다는 것 명시
    interface CoffeeMaker {
        makeCoffee(shots: number): CoffeeCup;
    }

    interface CommercialCoffeeMaker {
        makeCoffee(shots: number): CoffeeCup;
        fillCoffeeBeans(beans: number): void;
        clean(): void;
    }

    class CoffeeMachine implements CoffeeMaker, CommercialCoffeeMaker {
        private static BEANS_GRAMM_PER_SHOT: number = 7;
        private coffeeBeans: number = 0;

        private constructor(coffeeBenas: number) {
            this.coffeeBeans = coffeeBenas; // 해당 클래스 안에 있는 coffeeBeans를 전달된 coffeeBeans만큼 할당
        }   

        static makeMachine(coffeeBeans: number): CoffeeMachine {
            return new CoffeeMachine(coffeeBeans);
        }

        fillCoffeeBeans(beans: number) {
            if (beans < 0) {
                throw new Error('values for beans should be greater than 0');
            }
            this.coffeeBeans += beans;
        }

        clean() {
            console.log('cleaning the machine...🧼');
        }

        private grindBeans(shots: number) {
            console.log(`grinding beans for ${shots}`);
            if (this.coffeeBeans < shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT) {
                throw new Error('Not enough coffee beans!!');
            }
            this.coffeeBeans -= shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT;
        }

        private preheat(): void {
            console.log('heating up...🔥');
            
        }

        private extract(shots: number): CoffeeCup {
            console.log(`Pulling ${shots} shots...☕️`);
            return {
                shots,
                hasMilk: false,
            }
            
        }

        makeCoffee(shots: number): CoffeeCup {
            this.grindBeans(shots);
            this.preheat();
            return this.extract(shots);
        }
    }

    class AmateurUser {
        constructor(private machine: CoffeeMaker) {}
        makeCoffee() {
            const coffee = this.machine.makeCoffee(2);
            console.log(coffee);
            
        }
    }
    class ProBarista {
        constructor(private machine: CommercialCoffeeMaker) {}
        makeCoffee() {
            const coffee = this.machine.makeCoffee(2);
            console.log(coffee);
            this.machine.fillCoffeeBeans(45);
            this.machine.clean();
        }        
    }

    const maker: CoffeeMachine = CoffeeMachine.makeMachine(32);
    const amateur = new AmateurUser(maker);
    const pro = new ProBarista(maker);

    amateur.makeCoffee();
    pro.makeCoffee();
}

// 출력
// Amateur
grinding beans for 2
heating up...🔥
Pulling 2 shots...☕️
{ shots: 2, hasMilk: false }

// Pro
grinding beans for 2
heating up...🔥
Pulling 2 shots...☕️
{ shots: 2, hasMilk: false }
cleaning the machine...🧼
```

<aside>
💡 동일한 오브젝트의 인스턴스일지라도, 오브젝트는 두 가지의 인터페이스를 구현하므로 `AmateurUser`은 `CoffeMaker`을 생성자로부터 받아오고, `ProBarista`는 `CommercialCoffeeMaker`을 생성자로부터 받아오기 때문에 인터페이스에서 규약된 클래스보다는 좀 더 좁은 범위에, 인터페이스에서 규약된 함수들만 접근 가능하다.

</aside>

### 5️⃣ 상속으로 다양한 커피 기계 만들기 (Inheritance)

> 중복된 코드는 가독성이 매우 떨어지므로, 이를 상속을 통해 코드의 재사용성을 높이자.
> 

```tsx
{
    type CoffeeCup = {
        shots: number;
        hasMilk: boolean;
    }

    // 2. 인터페이스 정의를 통한 추상화
    // CoffeeMaker라는 인터페이스를 이용하면 makeCoffee를 이용할 수 있다는 것 명시
    interface CoffeeMaker {
        makeCoffee(shots: number): CoffeeCup;
    }

    class CoffeeMachine implements CoffeeMaker {
        private static BEANS_GRAMM_PER_SHOT: number = 7;
        private coffeeBeans: number = 0;

        constructor(coffeeBenas: number) {
            this.coffeeBeans = coffeeBenas; // 해당 클래스 안에 있는 coffeeBeans를 전달된 coffeeBeans만큼 할당
        }   

        static makeMachine(coffeeBeans: number): CoffeeMachine {
            return new CoffeeMachine(coffeeBeans);
        }

        fillCoffeeBeans(beans: number) {
            if (beans < 0) {
                throw new Error('values for beans should be greater than 0');
            }
            this.coffeeBeans += beans;
        }

        clean() {
            console.log('cleaning the machine...🧼');
        }

        private grindBeans(shots: number) {
            console.log(`grinding beans for ${shots}`);
            if (this.coffeeBeans < shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT) {
                throw new Error('Not enough coffee beans!!');
            }
            this.coffeeBeans -= shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT;
        }

        private preheat(): void {
            console.log('heating up...🔥');
            
        }

        private extract(shots: number): CoffeeCup {
            console.log(`Pulling ${shots} shots...☕️`);
            return {
                shots,
                hasMilk: false,
            }
            
        }

        makeCoffee(shots: number): CoffeeCup {
            this.grindBeans(shots);
            this.preheat();
            return this.extract(shots);
        }
    }

    class CaffeLatteMachine extends CoffeeMachine {
				// ⭐️ 
        constructor(beans: number, public readonly serialNumber: string) {
            super(beans);
        }
        private steamMilk(): void {
            console.log('Steaming some milk...🥛');
        }
				// ✏️
        makeCoffee(shots: number): CoffeeCup {
            const coffee = super.makeCoffee(shots);
            this.steamMilk();
            return {
                ...coffee,
                hasMilk: true
            }
        }
    }

    const machine = new CoffeeMachine(23);
    const latteMachine = new CaffeLatteMachine(23, 'SEIIQ12');
    const coffee = latteMachine.makeCoffee(1);
    console.log(coffee);
    console.log("라떼머신 번호", latteMachine.serialNumber);
}
```

<aside>
💡 interface 구현시 implements, class 구현시 extends를 사용하여 상속한다.

</aside>

<aside>
💡 자식 클래스에서 부모 클래스에 있는 함수를 덮어쓸 수 있다. (overwriting)
만약 부모 클래스에 있는 함수를 그대로 쓰고 싶다면, super 키워드를 사용하면 된다.

</aside>

> ✏️ **부모 클래스의 makeCoffee 함수의 매커니즘을 그대로 따르면서, 자식 클래스에 추가적으로 구현된 부분이 표현된다. 
즉, 위 코드에서는 CoffeeMachine 클래스의 makeCoffee 함수를 CaffeLatteMachine 클래스가 상속받은 후 steamMilk()를 추가하여 활용한 것이다.**
> 

<aside>
💡 이처럼 상속을 잘 이용하면, 공통적인 기능은 재사용하면서 자식 클래스에 특화된 기능을 추가할 수 있는 것이다.

</aside>

⭐️ **만약 자식클래스에서 생성자를 따로 구현하는 경우엔, 부모의 생성자도 호출해줘야 한다. 즉, 부모 클래스에서 필요한 데이터를 받아오고 받아온 데이터를 super()를 이용해 전달해줘야 한다.**

### 다형성 (Polymorphism)

> ✔︎ **하나의 인터페이스나 부모의 클래스를 상속한 자식 클래스들이 인터페이스와 부모 클래스에 있는 함수들을 다른 방식으로 다양하게 구성함으로써 다양성을 만드는 것을 말한다.

✔︎ 인터페이스와 부모 클래스에 있는 동일한 함수 api를 통해 각각 구현된 자식 클래스의 내부 구현 사항을 신경 쓰지 않고 약속된 한 가지의 api를 호출해서 사용자도 간편하게 다양한 기능들을 활용하도록 도와준다.

✔︎ 다형성을 이용하면 한 가지의 클래스나 인터페이스를 통해 다른 방식으로 구현한 클래스를 만들 수 있다.**
> 

**장점**

- **내부적으로 구현된 다양한 클래스들이 한가지의 인터페이스를 구현하거나 동일한 부모 클래스를 상속했을 때 동일한 함수를 어떤 클래스인지 구분하지 않고 공통된 api를 호출할 수 있다.**

```tsx
{
    type CoffeeCup = {
        shots: number;
        hasMilk?: boolean;
        hasSugar?: boolean;
    }

    interface CoffeeMaker {
        makeCoffee(shots: number): CoffeeCup;
    }

    class CoffeeMachine implements CoffeeMaker {
        private static BEANS_GRAMM_PER_SHOT: number = 7;
        private coffeeBeans: number = 0;

        constructor(coffeeBenas: number) {
            this.coffeeBeans = coffeeBenas; // 해당 클래스 안에 있는 coffeeBeans를 전달된 coffeeBeans만큼 할당
        }   

        static makeMachine(coffeeBeans: number): CoffeeMachine {
            return new CoffeeMachine(coffeeBeans);
        }

        fillCoffeeBeans(beans: number) {
            if (beans < 0) {
                throw new Error('values for beans should be greater than 0');
            }
            this.coffeeBeans += beans;
        }

        clean() {
            console.log('cleaning the machine...🧼');
        }

        private grindBeans(shots: number) {
            console.log(`grinding beans for ${shots}`);
            if (this.coffeeBeans < shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT) {
                throw new Error('Not enough coffee beans!!');
            }
            this.coffeeBeans -= shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT;
        }

        private preheat(): void {
            console.log('heating up...🔥');
            
        }

        private extract(shots: number): CoffeeCup {
            console.log(`Pulling ${shots} shots...☕️`);
            return {
                shots,
                hasMilk: false,
            }
            
        }

        makeCoffee(shots: number): CoffeeCup {
            this.grindBeans(shots);
            this.preheat();
            return this.extract(shots);
        }
    }

    class CaffeLatteMachine extends CoffeeMachine {
        constructor(beans: number, public readonly serialNumber: string) {
            super(beans);
        }
        private steamMilk(): void {
            console.log('Steaming some milk...🥛');
        }
        makeCoffee(shots: number): CoffeeCup {
            const coffee = super.makeCoffee(shots);
            this.steamMilk();
            return {
                ...coffee,
                hasMilk: true
            }
        }
    }

    class SweetCoffeeMaker extends CoffeeMachine {
        makeCoffee(shots: number): CoffeeCup {
            const coffee = super.makeCoffee(shots);
            return {
                ...coffee,
                hasSugar: true
            }
        }
    }

    // 부모 클래스인 CoffeeMachine이 CoffeeMaker 인터페이스의 규격에 따르기 때문에 
    // 두 자식 클래스는 CoffeeMaker 배열로 만들 수 있다.
    const machines: CoffeeMaker[] = [
        new CoffeeMachine(16),
        new CaffeLatteMachine(16, '1'),
        new SweetCoffeeMaker(16),
        new CoffeeMachine(16),
        new CaffeLatteMachine(16, '1'),
        new SweetCoffeeMaker(16)
    ]; 
    machines.forEach(machine => {
        console.log('----------------------');
        machine.makeCoffee(1);
    })
}

// 출력
----------------------
grinding beans for 1
heating up...🔥
Pulling 1 shots...☕️
----------------------
grinding beans for 1
heating up...🔥
Pulling 1 shots...☕️
Steaming some milk...🥛
----------------------
grinding beans for 1
heating up...🔥
Pulling 1 shots...☕️
----------------------
grinding beans for 1
heating up...🔥
Pulling 1 shots...☕️
----------------------
grinding beans for 1
heating up...🔥
Pulling 1 shots...☕️
Steaming some milk...🥛
----------------------
grinding beans for 1
heating up...🔥
Pulling 1 shots...☕️
```

### 👿 상속의 문제점

> **✔︎ 어떤 부모 클래스의 행동을 수정하면 해당 사항때문에 이를 상속하는 모든 자식 클래스에 영향을 미칠 수 있다.
✔︎ 새로운 기능 도입시 어떻게 상속의 구조를 지어야 할지 복잡하다.
✔︎ 타입스크립트에서는 한 가지 이상의 부모클래스를 상속할 수 없다.
→ Classes can only extend a single class**
> 

<aside>
💡 Favor Composition over inheritance 
: 상속보단 구성을 더 선호하라

</aside>

⭐️ 상속만을 이용해서 깊이있게 상속을 해버리면 관계가 복잡해질 수 있으므로 불필요한 상속 대신에 Composition을 이용해보자.

<aside>
💡 중복되는 부분은 외부에서 주입 받아서 가져와보자.
→ Dependency Injection

</aside>

<aside>
💡 `CaffeLatteMaker` 클래스의 `steamMilk()` 부분을 지우고 해당 부분을 다음과 같이 수정해준다.
`constructor`부분과 `makeCoffee()`의 return 부분에 변화가 생긴다.

</aside>

```tsx
// 싸구려 우유 거품기
class CheapMilkSteamer {
    private steamMilk(): void {
        console.log('Steaming some milk...🥛');
    }
    makeMilk(cup: CoffeeCup): CoffeeCup {
        this.steamMilk();
        return {
            ...cup,
            hasMilk: true,
        }
    }
}

class CaffeLatteMachine extends CoffeeMachine {
    constructor(
        beans: number, 
        public readonly serialNumber: string, 
        private milkFother: CheapMilkSteamer
        ) {
        super(beans);
    }
    makeCoffee(shots: number): CoffeeCup {
        const coffee = super.makeCoffee(shots);
        return this.milkFother.makeMilk(coffee);
    }
}
```

<aside>
💡 `SweetCoffeeMaker`도 왼쪽과 같이 수정해준다.

</aside>

```tsx
// 설탕 제조기
class AutomaticSugarMixer {
    private getSugar() {
        console.log('Getting some sugar from jar 🍭');
        return true;
    }

    addSugar(cup: CoffeeCup): CoffeeCup {
        const sugar = this.getSugar();
        return {
            ...cup,
            hasSugar: sugar,
        }
    }
}

class SweetCoffeeMaker extends CoffeeMachine {
    constructor(private beans: number, private sugar: AutomaticSugarMixer) {
        super(beans);
    };
    makeCoffee(shots: number): CoffeeCup {
        const coffee = super.makeCoffee(shots);
        return this.sugar.addSugar(coffee);
    }
}
```

<aside>
💡 위 방식이 바로, 각각의 기능별로 클래스를 따로 만들어, 필요한 곳에서 가져다 쓰는 Composition방식이다.

</aside>

🖇 **이 방식을 이용해서 SweetCaffeLatteMachine을 만들어보자!** 

```tsx
class SweetCaffeLatteMachine extends CoffeeMachine {
        constructor(
            private beans: number, 
            private milk: CheapMilkSteamer, 
            private sugar: AutomaticSugarMixer
            ){
                super(beans);
        }
        makeCoffee(shots: number): CoffeeCup {
            const coffee = super.makeCoffee(shots);
            const sugarAdded = this.sugar.addSugar(coffee);
            return this.milk.makeMilk(sugarAdded);
```

> ✔︎ 이와 같이 Composition을 이용해서 필요한 기능을 외부로부터 받아와 재사용할 수 있다. 
코드의 재사용성을 높여주는 것이다.

✔︎ 단점
: 위 클래스들은 `CheapMilkSteamer`와 `AutomaticSugarMixer`와 굉장히 타이트하게 커플링되어져 있다.
⭐️ 클래스 간에 연관되어 있는 것은 좋은 코드가 아니다!
> 

✍️ *여기서 말한 단점을 개선할만한 테크닉에 대해 아래에서 바로 언급하겠다.*

### 💪🏼 강력한 Interface

<aside>
💡 클래스 간 상호작용이 발생하는 경우, 클래스 자신을 노출하는게 아니라 interface를 통해 서로간의 상호작용을 해야 한다. (decoupling의 원칙)

</aside>

<aside>
💡 아래 코드에서 위에서 만든 모든 클래스는 지워주고 (기능 구현에 필요한 최소한의 인터페이스와 클래스 제외) 이를 `CoffeeMachine` 클래스를 다음과 같이 수정함으로써 코드의 재사용성을 높여준다.
인터페이스를 통해 상호작용하기 위해 `MilkFrother`, `SugarProvider`라는 인터페이스를 이용했다.

</aside>

```tsx
{
    type CoffeeCup = {
        shots: number;
        hasMilk?: boolean;
        hasSugar?: boolean;
    }

    interface CoffeeMaker {
        makeCoffee(shots: number): CoffeeCup;
    }

    class CoffeeMachine implements CoffeeMaker {
        private static BEANS_GRAMM_PER_SHOT: number = 7;
        private coffeeBeans: number = 0;

        constructor(
            coffeeBenas: number, 
            private milk: MilkFrother, 
            private sugar: SugarProvider
        ) 
        {
            this.coffeeBeans = coffeeBenas; // 해당 클래스 안에 있는 coffeeBeans를 전달된 coffeeBeans만큼 할당
        }

        fillCoffeeBeans(beans: number) {
            if (beans < 0) {
                throw new Error('values for beans should be greater than 0');
            }
            this.coffeeBeans += beans;
        }

        clean() {
            console.log('cleaning the machine...🧼');
        }

        private grindBeans(shots: number) {
            console.log(`grinding beans for ${shots}`);
            if (this.coffeeBeans < shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT) {
                throw new Error('Not enough coffee beans!!');
            }
            this.coffeeBeans -= shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT;
        }

        private preheat(): void {
            console.log('heating up...🔥');
            
        }

        private extract(shots: number): CoffeeCup {
            console.log(`Pulling ${shots} shots...☕️`);
            return {
                shots,
                hasMilk: false,
            }
            
        }

        makeCoffee(shots: number): CoffeeCup {
            this.grindBeans(shots);
            this.preheat();
            const coffee = this.extract(shots);
            const sugarAdded = this.sugar.addSugar(coffee);
            return this.milk.makeMilk(sugarAdded);
        }
    }

    interface MilkFrother {
        makeMilk(cup: CoffeeCup): CoffeeCup;
    }

    interface SugarProvider {
        addSugar(cup: CoffeeCup): CoffeeCup;
    }

    // 싸구려 우유 거품기
    class CheapMilkSteamer implements MilkFrother {
        private steamMilk(): void {
            console.log('Steaming some milk...🥛');
        }
        makeMilk(cup: CoffeeCup): CoffeeCup {
            this.steamMilk();
            return {
                ...cup,
                hasMilk: true,
            }
        }
    }

    class FancyMilkSteamer implements MilkFrother {
        private steamMilk(): void {
            console.log('Fancy Steaming some milk...🥛');
        }
        makeMilk(cup: CoffeeCup): CoffeeCup {
            this.steamMilk();
            return {
                ...cup,
                hasMilk: true,
            }
        }
    }

    class ColdMilkSteamer implements MilkFrother {
        private steamMilk(): void {
            console.log('Fancy Steaming some milk...🥛');
        }
        makeMilk(cup: CoffeeCup): CoffeeCup {
            this.steamMilk();
            return {
                ...cup,
                hasMilk: true,
            }
        }
    }

    // 우유를 만들지 않음
    class NoMilk implements MilkFrother {
        makeMilk(cup: CoffeeCup): CoffeeCup {
            return cup;
        }
    }

    // 설탕 제조기
    class CandySugarMixer implements SugarProvider {
        private getSugar() {
            console.log('Getting some sugar from jar 🍭');
            return true;
        }

        addSugar(cup: CoffeeCup): CoffeeCup {
            const sugar = this.getSugar();
            return {
                ...cup,
                hasSugar: sugar,
            }
        }
    }

    class SugarMixer implements SugarProvider {
        private getSugar() {
            console.log('Getting some sugar from jar 🍭');
            return true;
        }

        addSugar(cup: CoffeeCup): CoffeeCup {
            const sugar = this.getSugar();
            return {
                ...cup,
                hasSugar: sugar,
            }
        }
    }

    class NoSugar implements SugarProvider {
        addSugar(cup: CoffeeCup): CoffeeCup {
            return cup;
        }
    }

    

    // Milk
    const cheapMilkMaker = new CheapMilkSteamer();
    const fancyMilkMaker = new FancyMilkSteamer();
    const coldMilkMaker = new ColdMilkSteamer();
    const noMilk = new NoMilk();

    // Sugar
    const candySugar = new CandySugarMixer();
    const sugar = new SugarMixer();
    const noSugar = new NoSugar();

    // 
    const sweetCandyMahcine = new CoffeeMachine(12, noMilk, candySugar);
    const sweetMahcine = new CoffeeMachine(12, noMilk, sugar);

    const latteMachine = new CoffeeMachine(12, cheapMilkMaker, noSugar);
    const coldLatteMachine = new CoffeeMachine(12, coldMilkMaker, noSugar);
    const sweetLatteMachine = new CoffeeMachine(12, cheapMilkMaker, candySugar);
}
```

> 상속이 무조건 나쁘고, composition만 이용해야하는 건 아니지만, 코드가 너무 수직적인 관계 (깊은 관계)라면 composition을 이용해서 좀 더 필요한 기능들을 조립해서 확장이 가능하고 재사용성이 높고, 유지보수가 쉬우며 더 높은 퀄리티의 코드를 만들기 위해 고민하는 과정이 중요하다.
> 

<aside>
💡 기능을 구현하는 게 우선이지, 확장성만 고려해서 코드를 복잡하게 짤 필요는 없다.

</aside>

### ✏️ Abstract 클래스

> ✔︎ abstract 클래스 자체는 object 생성이 불가능하다. (추상적인 클래스)
→ 공통적인 기능 구현 가능

✔︎ 구현하는 클래스마다 달라져야하는 부분이 있다면 해당 부분만 abstract 메소드로 정의한다.
→ 함수 이름이 무엇인지, 인자를 무엇을 받아서 무엇을 리턴하는지만 정의할 수 있다.
→ `protected abstract extract()`
> 

```tsx
// abstract 키워드를 붙이면 CoffeeMachine 자체로는 Object 생성 불가
abstract class CoffeeMachine implements CoffeeMaker {
    private static BEANS_GRAMM_PER_SHOT: number = 7;
    private coffeeBeans: number = 0;

    constructor(coffeeBenas: number) {
        this.coffeeBeans = coffeeBenas; // 해당 클래스 안에 있는 coffeeBeans를 전달된 coffeeBeans만큼 할당
    }

    fillCoffeeBeans(beans: number) {
        if (beans < 0) {
            throw new Error('values for beans should be greater than 0');
        }
        this.coffeeBeans += beans;
    }

    clean() {
        console.log('cleaning the machine...🧼');
    }

    private grindBeans(shots: number) {
        console.log(`grinding beans for ${shots}`);
        if (this.coffeeBeans < shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT) {
            throw new Error('Not enough coffee beans!!');
        }
        this.coffeeBeans -= shots * CoffeeMachine.BEANS_GRAMM_PER_SHOT;
    }

    private preheat(): void {
        console.log('heating up...🔥');
        
    }

    // abstract 메소드는 구현사항을 쓰면 안됨.
    protected abstract extract(shots: number): CoffeeCup ;

    makeCoffee(shots: number): CoffeeCup {
        this.grindBeans(shots);
        this.preheat();
        return this.extract(shots);
    }
}

class CaffeLatteMachine extends CoffeeMachine {
    constructor(beans: number, public readonly serialNumber: string) {
        super(beans);
    }
    private steamMilk(): void {
        console.log('Steaming some milk...🥛');
    }
		// abstract method 사용
    protected extract(shots: number): CoffeeCup {
        this.steamMilk();
        return {
            shots,
            hasMilk: true,
        }
    }
}
```

<aside>
💡 **abstract 클래스를 이용하면 공통적인 기능들을 수행하고 달라져야 하는 부분만 상속하는 클래스에게 강조할 수 있는 효과가 있다.**

</aside>
