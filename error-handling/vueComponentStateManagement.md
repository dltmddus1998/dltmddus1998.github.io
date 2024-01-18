# Vue3 & HTML - 특정 요소 잔상 문제

## Why - 컴포넌트 상태 관리

leader-line-new라는 특정 요소간 선을 그려줄 수 있는 라이브러리가 있어서 토폴로지 구현 중 활용해보았다. 

그러던 중, 다른 페이지로 넘어갔을 때 **해당 선들이 계속 잔상으로 남아있는 현상**이 있었고 chatGPT에 질의 해보니 다음과 같이 말했다.

> **라이브러리나 컴포넌트 간의 상태 관리 문제로 보이며, 일반적으로 페이지 전환 시에 이러한 문제가 발생한다면,
Vue 인스턴스나 컴포넌트의 소멸과 함께 해당 라이브러리에서 사용한 자원이 정리되지 않아 발생하는 것일 수 있습니다.**
> 

## How - `onBeforeRouteLeave()` 사용

그래서 이에 대해 리서치 해보니 `onBeforeRouteLeave`라는 vue에 내장되어있는 메서드를 활용하면 된다고 한다. 

`onBeforeRouteLeave`는 말 그대로 현재 페이지에 해당하는 라우트를 벗어나기 직전에 일어날 액션에 대해 구현하면 된다.

그래서 나는 다음과 같이 구현했다.

```jsx
onBeforeRouteLeave((to, from, next) => {
  leaderLineInstances.forEach((line: LeaderLine) => {
    console.log(line);
    line.remove();
  });
  next();
});
```

>원래 코드의 극히 일부분인데 간략히 설명하자면, leaderLineInstances라는 ref형태의 배열인데 여기 안에 leader-line-new로 만든 선들을 모두 push해놨다. 

>이제 이걸 forEach를 통해 하나씩 remove해주는 형태이다. 다행히 해당 코드는 잘 적용되었고, 이제 다른 페이지로 넘어가도 잔상이 남지 않는다.
