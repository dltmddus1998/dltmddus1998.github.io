# ๐ง interface์ type์ ์ฐจ์ด์ 

## interface๋ ๊ฐ์ฒด์๋ง ์ฌ์ฉ ๊ฐ๋ฅ

```tsx
interface FooInterface {
	value: string 
}

type FooType = {
	value: sring
}

type FooOnlyString = string
type FooTypeNumber = number

// ๋ถ๊ฐ๋ฅ
interface X extends string {}
```

## computed value์ ์ฌ์ฉ

> `type`์ ๊ฐ๋ฅํ์ง๋ง `interface`๋ ๋ถ๊ฐ๋ฅ
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

## ์ฑ๋ฅ์ interface is better!

๊ด๋ จ ๋ฌธ์ ์ฐธ์กฐ ([https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections](https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections))

> ์ฌ๋ฌ `type` ํน์ `interface`๋ฅผ `&`ํ๊ฑฐ๋ `extends`ํ  ๋ `interface`๋ โ์์ฑ ๊ฐ ์ถฉ๋์ ํด๊ฒฐํ๊ธฐโ ์ํด โ๋จ์ํ ๊ฐ์ฒด ํ์์ ๋ง๋ ๋คโ. 
โ `interface`๋ โ๊ฐ์ฒด์ ํ์์ ๋ง๋ค๊ธฐ ์ํ ๊ฒโ์ด๋ฏ๋ก ๊ฐ์ฒด๋ฅผ ๋จ์ํ ๋จธ์งํ๊ธฐ๋ง ํ๋ฉด ๋๋ฏ๋ก.
> 

> BUT, `type`์ โ์ฌ๊ท์ ์ผ๋ก ์ํํ๋ฉด์ ์์ฑ์ ๋จธ์งโํ๋๋ฐ, ์ด ๊ฒฝ์ฐ ์ผ๋ถ `never`๊ฐ ๋์ค๋ฉด์ ์ ๋๋ก ์คํ๋์ง ์๋๋ค.
> 

<aside>
๐ก ์ฆ, `interface`์๋ ๋ฌ๋ฆฌ `type`์ ์์ ํ์์ด ์ฌ ์๋ ์์ด์ ์ถฉ๋์ด ๋๋ฉด ์ ๋๋ก ๋จธ์ง๊ฐ ๋์ง ์์ ์ ์๋ค.

</aside>

```tsx
type type2 = { a: 1 } & { b: 2 } // ok
type type3 = { a: 1; b: 2 } & { b: 3 } // resolved to never

const t2: type2 = { a: 1, b: 2 } // good
const t3: type3 = { a: 1, b: 3 } // Type 'number' is not assignable to type 'never'.(2322) 
const t3: type3 = { a: 1, b: 2 } // Type 'number' is not assignable to type 'never'.(2322)
```

โ๏ธ ๊ฒฐ๊ตญ, ๊ฐ์ฒด์์๋ง ์ฐ๋ ์ฉ๋๋ผ๋ฉด interface๋ฅผ ๋๊ณ  type์ ์ธ ์ด์ ๊ฐ ์ ํ ์๋ค.

# ๐ก ๊ฒฐ๋ก : ๊ฐ์ฒด, ๊ทธ๋ฆฌ๊ณ  ํ์ ๊ฐ์ ํฉ์ฑ๋ฑ์ ๊ณ ๋ คํ์ ๋ interface๋ฅผ ์ฐ๋ ๊ฒ์ด ๋ซ๋ค.
