# Vue - Reactivity Fundamentals

## Declaring Reactive State

> Reactive 객체들은 JavaScript Proxy들이며, 일반적인 객체처럼 동작한다. 차이는 Vue가 속성 접근 및 반응 객체의 돌연변이를 추적할 수 있다는 점이다.
> 

setup() 구성 요소의 템플렛에서 반응 상태를 사용하려면 구성 요소의 함수에서 선언하고 반환한다.

```tsx
<template>
  <div>{{ state }}</div>
</template>

<script>
import { reactive } from 'vue';

export default {
  setup() {
    const state = reactive({ count: 0 });

    return {
      state,
    };
  },
};
</script>
```

마찬가지로 동일한 범위에서 반응 상태를 변경하는 함수를 선언하고 상태와 함께 메스드로 노출할 수 있다.

```tsx
<template>
  <button @click="increment">{{ state.count }}</button>
</template>

<script>
import { reactive } from 'vue';

export default {
  setup() {
    const state = reactive({ count: 0 });

    function increment() {
      state.count++;
    }

    return {
      state,
      increment
    };
  },
};
</script>
```

<aside>
💡 API 기본 설정을 Option이 아닌 Composition으로 구현하면 다음과 같다.

</aside>

```tsx
<script setup>
import { reactive } from 'vue';

const state = reactive({ count: 0 })

function increment() {
  state.count++;
}

</script>

<template>
  <div>{{ state }}</div>
  <button @click="increment">{{ state.count }}</button>
</template>
```

### DOM Update Timing

> Reactive State을 변형할 때 DOM은 자동적으로 업데이트되지만, DOM 업데이트들은 동기적으로 지원되지 않다는 것을 알아야 한다. 대신, Vue는 업데이트 주기의 “next tick”까지 버퍼링하여 상태 변경 횟수에 관계 없이 각 구성 요소가 한 번만 업데이트 되도록 한다.
> 

> 상태 변화 이후 DOM 업데이트가 완료되기를 기다리기 위해 `nextTick()` 이라는 global API를 사용할 수 있다.
> 

```tsx
<script setup>
import { reactive, nextTick } from 'vue';

const state = reactive({ count: 0 })

function increment() {
  state.count++;
  nextTick(() => {
    // access updated DOM
  })
}

</script>
```

### Deep Reactivity

> Vue에서 상태는 기본적으로 매우 reactive하다.
즉, 중첩된 객체나 배열을 변경하는 경우에도 변경 사항이 감지될 것으로 예상할 수 있다.
> 

```tsx
<script setup>
import { reactive } from 'vue';

const obj = reactive({
  nested: { count: 0 },
  arr: ['foo', 'bar'],
});

function mutateDeeply() {
  // these will work as expected
  obj.nested.count++;
  obj.arr.push('baz');
}
</script>
```

### Reactive Proxy vs. Original

> `reactive()`로부터 반환된 값이 원래의 객체와 동일하지 않은 원래의 객체의 Proxy라는 사실은 매우 중요하다.
> 

```tsx
const raw = {}
const proxy = reactive(raw)

// proxy is NOT equal to the original
console.log(proxy === raw) // false 
```

![image](https://user-images.githubusercontent.com/73332608/199658705-2ac204e8-841d-45c3-88df-65fca50997b8.png)

위와 같이 실제로 reactive로부터 반환된 값은 Proxy로 감싸진 값이다.

> **Reactive한건 Proxy 뿐이다.** 
→ 원래의 객체를 변형하는 것이 업데이트를 유발하지는 않을 것이다.
> 

> 따라서, Vue의 Reactive System으로 작업할 때 최선의 방식은 해당 상태의 proxy된 버전을 사용하는 것이다.
> 

> Proxy로의 지속적인 접근을 보장하기 위해 같은 객체에서 reactive()를 호출하면 항상 동일한 Proxy로 반환되고 기존 Proxy에 대해서도 동일한 Proxy가 반환된다.
> 

```tsx
// calling reactive() on the same object returns on the same proxy
console.log(reactive(raw) === proxy) // true

// calling reactive() on a proxy returns itself
console.log(reactive(proxy) === proxy) // true 
```

> 해당 규칙은 중첩된 객체들 또한 지원한다. Deep Reactivity 때문에 Reactive한 객체 안에 있는 중첩된 객체들 또한 Proxy들이다.
> 

```tsx
const proxy = reactive({})

const raw = {}
proxy.nested = raw;

console.log(proxy.nested === raw) // false 
```

### Limitations of reactive()

**reactive() API는 두 가지 한계점을 가지고 있다.**

1. reactive()는 객체 타입에만 적용된다. (객체, 배열, 그리고 Map과 Set과 같은 컬렉션 타입)
    
    → string, number, boolean과 같은 원시 타입은 적용할 수 없다.
    
2. Vue의 Reactivity는 **속성 접근에 대해 작동**하기 때문에, Reactive한 객체에 대해 항상 동일한 참조를 유지해야한다. 
    
    → 첫번째 참조에 대한 Reactivity 연결은 손실이기 때문에 Reactive한 객체를 쉽게 대체할 수 없다. 
    

```tsx
let state = reactive({ count: 0 });

// the above reference ({ count: 0 }) is no longer being tracked 
// reactivity connection is lost!!
state = reactive({ count: 0 });
```

이는 Reactive한 객체의 속성을 로컬한 값들에 할당하거나 구조를 해제할 때, 또는 함수에서의 속성을 넘길 때 Reactivity 연결이 손실될 것이라는 것을 의미한다. 

```tsx
const state = reactive({ count: 0 });

// n is a local variable that is disconnected from state.count
let n = state.count;
// does not affect original state
n++;

// count is also disconnected from state.count
let { count } = state;
// does not affect original state
count++;

// the function receives a plain number and 
// won't be able to track changes to state.count
callSomeFunction(state.count)
```

💡 **즉, reactive로 반환된 값 내의 속성들을 변경한다 하더라도 원래의 상태가 변하지는 않는다.**

## Reactive Variables with ref()

> 위에서 언급한 reactive()의 한계점들을 보완하기 위해 Vue는 어느 value 타입이든 다룰 수 있는 reactive한 “refs”를 만들 수 있는 `ref()` 함수 또한 제공한다.
> 

```tsx
import { ref } from 'vue';

const count = ref(0);
```

> `ref()`는 인자를 가지고 `.value` 라는 속성과 함께 ref 객체 안에 감싸진 상태로 반환된다.
> 

```tsx
const count = ref(0);

console.log(count); // { value: 0 }
console.log(count.value); // 0

count.value++;
console.log(count.value); // 1
```

### Typing ref()

> Ref는 초기값을 통해 타입을 추론한다.
> 

```tsx
import { ref } from 'vue';

// inferred type: Ref<number>
const year = ref(2020)

// => TS Error: Type 'string' is not assignable to type 'number'
year.value = '2020';
```

> ref 내부 값에 대해 특정한 복잡한 타입을 필요로 할 때가 있다. 
이 때 Ref 타입을 활용할 수 있다.
> 

```tsx
import { ref } from 'vue';
import type { Ref } from 'vue';

const year: Ref<string | number> = ref('2020');

year.value = 2020; // ok!
```

> 또는 ref()를 호출할 때 제네릭 인자를 사용할 수 있다.
> 

```tsx
// resulting type: Ref<string | number>
const year = ref<string | number>('2020');

year.value = 2020; // ok!
```

> 제네릭 타입의 인자를 명시했는데 초기값을 누락했다면, 결과값의 타입은 undefined를 포함한 union 타입일 것이다.
> 

```tsx
// inferred type: Ref<number | undefined>
const n = ref<number>()
```

> Reactive한 객체에 있는 속성들과 비슷하게, ref의 `.value` 속성 또한 reactive하다.
> 

> 또한, 객체 타입들을 보유할 때 ref는 자동으로 `.value`를 `reactive()`로 변환한다.
> 

> 객체를 포함하는 ref는 reactive하게 전체 객체를 대체할 수 있다.
> 

```tsx
const objectRef = ref({ count: 0 });

// this works reactively
onjectRef.value = { count: 1 };
```

> Ref는 또한 reactivity를 잃지 않은 채 함수로 전달되거나 일반 객체에서 구조화될 수 있다.
> 

```tsx
const obj = {
	foo: ref(1),
	bar: ref(2)
};

// the function receives a ref
// it needs to access the value via .value
// but it will retain the reactivity connection 
callSomeFunction(obj.foo)

// still reactive
const { foo, bar } = obj;
```

> 즉, ref()는 어느 값이든 “reference”를 만들게 해주고 reactivity의 손실 없이도 전달될 수 있도록 한다.
> 

> Composable Functions에서 로직을 추출할 때 빈번히 쓰이기 때문에 해당 기능은 꽤 중요하다
> 
- **Composable Functions란?**
    
    Vue 어플리케이션의 맥락을 보면, “composable”이란 Vue의 구성 API를 활용하여 상태 저장 로직을 캡슐화하고 재사용하기 위한 함수이다.
    
    프론트엔드 애플리케이션을 빌드할 때 주로 흔한 태스크들에 대한 로직을 재활용할 필요가 있는데 예를 들어, 많은 곳에서 날짜 형식을 만들어야 해서 그것을 위해 재사용이 가능한 함수를 추출해낸다. 이런 함수는 저장할 수 없는 상태 로직을 캡슐화한다: 몇몇의 input 값을 받고 즉시 예상 값을 반환한다. 
    
    대조적으로, 상태 저장 로직은 시간에 따라 상태를 관리하는 것을 포함한다. 간단한 예제로, 페이지의 마우스 현재 위치를 추적하는 것이다. 실제 시나리오에서, 이는 터치 제스처나 데이터베이스의 연결 상태와 같이 더 복잡한 로직일 수도 있다. 
    

### Ref Unwrapping in Templates

> ref가 template에서 상위 속성에 접근할 때, ref는 자동적으로 “unwrapped”돼서 `.value`를 사용할 필요가 없어진다.
> 

```html
<script setup>
import { ref } from 'vue';

const count = ref(0);

function increment() {
	count.value++;
}
</script>

<template>
	<button @click="increment">
		{{ count }} <!-- no .value needed -->
	</button>
</template>
```

> unwrapping은 ref가 template render 코드의 상위 속성일 때만 지원한다는 것을 기억해라.
> 

```tsx
const object = { foo: ref(1) }
```

다음 표현은 예상대로 동작하지 않을 것이다.

```tsx
{{ object.foo + 1 }}
```

> 렌더링된 결과는 `[object Object]`일 것이다. 왜냐하면, `object.foo`가 ref의 객체이기 때문이다.
> 

이는 `foo`의 상위 속성을 만듦으로써 해결할 수 있다.

```tsx
const { foo } = object;
.
.
.
{{ foo + 1 }}
```

### Ref unwrapping in Reactive Objects

> ref가 reactive한 객체의 속성으로 접근되거나 변형될 때, ref는 자동적으로 unwrap되기 때문에 일반적인 속성처럼 행동할 수 있다.
> 

```tsx
const count = ref(0);
const state = reactive({
	count
});

console.log(state.count); // 0

state.count = 1;
console.log(state.count); // 1
```

> 만약 새로운 ref가 이미 존재하는 ref에 포함된 속성에 할당된다면, 원래 있던 ref를 대체할 것이다.
> 

```tsx
const otherCount = ref(2);

state.count = otherCount;
console.log(state.count); // 2
// original ref is now disconnected from state.count
console.log(count.value); // 1
```

> Ref unwrapping은 매우 reactive한 객체 안에 중첩됐을 때만 일어난다.
이는 얕은 reactive 객체의 속성에 접근했을 때는 지원하지 않는다.
> 

### Ref Unwrapping in Arrays and Collections

reactive 객체들과는 달리 ref가 reactive한 배열의 요소나 Map과 같은 원시적인 컬렉션 타입에 접근할 때는 unwrapping이 발생하지 않는다.

```tsx
const books = reactive([ref('Vue 3 Guide')]);
// need .value here
console.log(books[0].value);

const map = reactive(new Map([['count', ref(0)]]));
// need .value here
console.log(map.get('count').value);
```

## Reactivity Transform

> ref와 함께 `.value`를 사용해야하는 점은 자바스크립트의 제약으로 인한 단점이다.
> 

> 그러나 컴파일 타임 변환을 사용하면 `.value`를 적절한 위치에 자동으로 추가하여 이를 개선할 수 있다.
> 

Vue는 컴파일 타임 변환을 제공한다.

```html
<script setup>
let count = $ref(0);

function increment() {
	// no need for .value
	count++;
}
</script>

<template>
	<button @click="increment">{{ count }}</button>
</template>
```
