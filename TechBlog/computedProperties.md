# Vue - Computed Properties

## Basic Examples

> template 내 표현식들은 매우 편리하지만 간단한 작업을 위한 것이다.
template 내에 너무 많은 로직을 넣는 것은 유지하기 어렵게 만들 수 있다.
> 

다음과 같이 객체를 중첩된 배열과 함게 갖는 경우가 있다.

```tsx
const author = reactive({
	name: 'John Doe',
	books: [
		'Vue 2 - Advanced Guide',
		'Vue 3 - Basic Guide',
		'Vue 4 - The Mystery',
	]
});
```

위의 코드에서 저자가 책이 있는지 없는지 확인하는 다른 메시지들을 표시하려한다.

```tsx
<p>Has published books:</p>
<span>{{ author.books.length > 0 ? 'Yes' : 'No' }}</span>
```

> 이 점에서, template은 복잡해진다.  author.books에 따라 계산을 수행한다는 것을 깨닫기 전에 template을 잠시 살펴봐야한다. 
더욱 중요한 것은, 아마 template내 계산을 한 번 이상 반복하고 싶지 않다는 점이다.
> 

```html
<script setup>
import { reactive, computed } from 'vue';

const author = reactive({
	name: 'Jogn Doe',
	books: [
		'Vue 2 - Advanced Guide',
		'Vue 3 - Basic Guide',
		'Vue 4 - The Mystery',
	]
});

<!-- a computed ref -->
const publishedBooksMessage = computed(() => {
	return author.books.length > 0 ? 'Yes' : 'No'
})
</script>

<template>
	<p>Has published books:</p>
	<span>{{ publishedBooksMessage }}</span>
</template>
```

> computed 속성인 `publishedBooksMessage`를 보면, `computed()` 함수는 getter 함수로 전달될 것으로 보이고 반환된 값은 computed ref이다.
> 

> 일반적인 ref와 마찬가지로, 해당 값은 `.value`를 통해 접근이 가능하고 template내에서는 unwrapping되므로 `.value`를 생략해도 된다.
> 

> computed property는 자동적으로 reactive한 의존성들을 추적한다. 
Vue는 `publishedBookdsMessage`의 계산이 `author.books`에 의존하는 것을 알기 때문에 `author.books`가 바뀔 때 `publishedBooksMessage`에 의지하는 모든 바인딩을 업데이트할 것이다.
> 

## Computed Caching vs. Methods

```html
<p>{{ calculateBooksMessage() }}</p>
```

```tsx
// in component
function calculateBooksMessage() {
	return author.books.length > 0 ? 'Yes' : 'No'
}
```

> computed property 대신, 메서드와 동일한 기능을 정의할 수 있다.
> 

💡 **결과적으로 두 가지 접근 방식은 동일하지만 computed property가 reactive한 종속성을 기반으로 캐시된다는 점에서 차이가 있다.**

> computed property는 reactive한 종속성에 변화가 있을 경우에만 재평가된다.
→ 따라서, `author.books`이 변하지 않는 한 `publishedBooksMessage`에 대한 다수의 접근은 getter 함수를 다시 실행시킬 필요 없이 계산된 결과값을 즉시 반환한다.
> 

```tsx
const now = computed(() => Date.now())
```

Date.now()는 reactive한 종속성이 아니므로 아래 예시 또한 다음 computed property는 절대 업데이트되지 않음을 알 수 있다.

✔️ **그에 반해, 메서드 호출은 리렌더링이 발생할 때마다 일어난다.**

> **캐싱이 필요한 이유는 무엇일까?**
값비싼 computed property인 list가 있고 그 list가 거대한 배열을 반복하고 많은 계산을 하는 것을 요구한다고 생각해보자. 그러면 list에 의존하는 computed property들이 있을 것이다. 캐싱이 없으면 list의 getter보다 필요 이상으로 많이 실행될 것이다. 캐싱을 원하지 않는 경우, 대신 메서드를 사용하면 된다.
> 

## Writable Computed

> computed property는 기본적으로 getter 전용이다. 
→ computed property에 새 값을 할당하려고 하면, 런타임 경고가 뜰 것이다.
> 

드물게 **쓸 수 있는** computed property가 필요한 경우, getter와 setter 둘 모두 제공하는 property를 생성할 수 있다.

```tsx
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  // getter
  get() {
    return firstName.value + ' ' + lastName.value
  },
  // setter
  set(newValue) {
    // Note: we are using destructuring assignment syntax here.
    [firstName.value, lastName.value] = newValue.split(' ')
  }
})
</script>
```
