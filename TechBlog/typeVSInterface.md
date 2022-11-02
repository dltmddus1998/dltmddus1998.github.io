# 🧐 interface와 type의 차이점

## interface는 객체에만 사용 가능

```tsx
interface FooInterface {
	value: string 
}

type FooType = {
	value: sring
}

type FooOnlyString = string
type FooTypeNumber = number

// 불가능
interface X extends string {}
```

## computed value의 사용

> `type`은 가능하지만 `interface`는 불가능
> 

```tsx
type names = 'firstName' | 'lastName'

type NameTypes = {
	[key in names]: string
}

const yc: NameTypes = { firstName: 'hi', lastName: 'yc' }

interface NameInterface {
	// error
	[key in names]: string
}
```

## 성능상 interface is better!

관련 문서 참조 ([https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections](https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections))

> 여러 `type` 혹은 `interface`를 `&`하거나 `extends`할 때 `interface`는 “속성 간 충돌을 해결하기” 위해 “단순한 객체 타입을 만든다”. 
→ `interface`는 “객체의 타입을 만들기 위한 것”이므로 객체를 단순히 머지하기만 하면 되므로.
> 

> BUT, `type`은 “재귀적으로 순회하면서 속성을 머지”하는데, 이 경우 일부 `never`가 나오면서 제대로 실행되지 않는다.
> 

<aside>
💡 즉, `interface`와는 달리 `type`은 원시 타입이 올 수도 있어서 충돌이 나면 제대로 머지가 되지 않을 수 있다.

</aside>

```tsx
type type2 = { a: 1 } & { b: 2 } // ok
type type3 = { a: 1; b: 2 } & { b: 3 } // resolved to never

const t2: type2 = { a: 1, b: 2 } // good
const t3: type3 = { a: 1, b: 3 } // Type 'number' is not assignable to type 'never'.(2322) 
const t3: type3 = { a: 1, b: 2 } // Type 'number' is not assignable to type 'never'.(2322)
```

✔️ 결국, 객체에서만 쓰는 용도라면 interface를 두고 type을 쓸 이유가 전혀 없다.

# 💡 결론: 객체, 그리고 타입 간의 합성등을 고려했을 때 interface를 쓰는 것이 낫다.
