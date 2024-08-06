---
title: "Vue에서 API 연결하기 (2)"
tags:
  - vue
  - axios
  - vueuse
date: "2024-08-06"
thumbnail: "/assets/img/thumbnail/nightgardenflower.jpg"
bookmark: true
---

# Vue에서 API 연결하기 (2)

## [Advanced] vue에서 axios만 사용하는게 더 편리하지 않을까?

> [axios 대신 vueuse의 useAxios를 사용한다면?](https://www.notion.so/axios-vueuse-useAxios-9faf71a408c244f3b9c2c02b6b63536d?pvs=21) 에 대해 FE 팀원과 스터디를 했는데 여러가지 의문에 대해 질문해주셔서 쓰게된 글이다.

### 질문 및 나의 생각

---

> ⁇: 현재 코드 상태를 봐서는 (global instance는 axios를 통해 create하고, 실제 api 함수는 useAxios로 구현하는 코드) 굳이 그렇게 구현해야 할 이유를 모르겠다. 그냥 axios만 쓰는게 더 간단하지 않을까?

> 💡: 솔직히 말하면 공식문서에 그렇게 써서, 그냥 문서상으로 useAxios를 쓰면 반응형으로 사용하기 용이하며, 추후 확장성을 생각했을 때 용이하다고 써져있었기 때문에 막연하게 쓰면 좋겠다! 라고 생각한게 큰 것 같다. 좀 더 깊게 생각하고 실제로 axios와 비교 테스트할 필요가 있다.

### 그래서?

---

> 우선 원래 코드에 덧붙여서 axios와 비교하기로 했다.

- useAxios와 axios 모두 사용해볼 필요도 없이 useAxios 부분에서 하나 막히는게 나왔다.
  - 먼저, useAxios의 이점은 반응형으로 사용하기 용이한 것이다. 즉 결과값 자체가 ref, reactive로 감싸져서 나오는데 이를 따로 모듈로 관리하려면 import해서 사용해야 하는 상황이 온다.
  - 이 때 import해서 data를 그대로 반응형으로 쓰려면 결국 똑같이 ref, reactive를 써야하기 때문에 반응형에 있어서 useAxios의 장점이 희미해진다.
- isLoading, isFinished 등 현재 api 통신 상태를 가져오기에 용이해서 로딩과 같은 애니메이션을 사용할 때 axios보다 쉽다는 장점도 있다.

### 결론

---

**추후의 확장성과 유연성을 생각한다면 vue 프레임워크에 해당하는 vueuse의 useAxios를 사용하는게 이점이 될 수 있다.**

**api를 따로 모듈화해서 사용하는 우리 프로젝트의 경우 어차피 가져와서 반응형 변수에 또 할당해야하기 때문에 반응형에 있어서는 이점이 되지 않는다.**

**따라서 현재는 `useAxios`를 굳이 쓸 필요가 없다.**
