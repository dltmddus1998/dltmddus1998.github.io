---
title: 'Vue에서 API 연결하기 (1)'
tags:
  - vue
  - axios
  - vueuse
date: '2024-08-06'
thumbnail: '/assets/img/La-Mancha.jpg'
bookmark: true
---

# Vue에서 API 연결하기 (1)

## axios 대신 vueuse의 useAxios를 사용한다면?

> **useAxios는 Vue의 유틸리티 라이브러리인 VueUse에서 제공하는 훅으로, Axios와 결합하여 더 편리한 API 호출 관리를 돕는다.**

### useAxios의 장﹒단점

- **장점**
  - 반응형 데이터 관리
    - useAxios는 Vue의 반응형 시스템과 잘 통합되어, API 호출 결과를 반응형으로 관리할 수 있음.
      → Vue의 ref, reactive와 같은 기능과 결합해 상태 관리가 자연스럽게 이루어짐.
    - ex) API 응답 데이터를 화면에 실시간으로 반영하거나 로딩 상태 등을 쉽게 추적 가능.
  - 편리한 상태 추적
    - useAxios는 로딩 상태, 에러 상태, 응답 데이터 등을 관리할 수 있는 상태값 제공.
      : API 호출의 현 상태 파악 용이 → UI에서 로딩 스피터 표시, 에러메시지 출력 등 구현 용이.
    - 상태값을 컴포넌트 내에서 간편하게 사용 가능 → 코드 가독성 up.
  - 자동 취소 기능
    - 컴포넌트가 언마운트되거나 특정 조건에서 불필요한 API 호출을 자동으로 취소할 수 있는 기능 제공.
      → 불필요한 네트워크 요청 감소 & 자원 효율적 사용 가능.
  - 타입스크립트 지원
    - useAxios는 타입스크립트와 효환성이 좋음.
      → 요청과 응답의 타입을 명확하게 정의할 수 있어, 타입 안정성 유지하며 개발 가능.
- **단점**
  - 추가 학습 곡선
    - useAxios를 사용하려면 VueUse 라이브러리와 그 API에 대한 학습 필요
  - 의존성 추가
    - vue에 대한 프로젝트 의존성 증가 → 프로젝트의 빌드 크기 증가나 업데이트시 호환성 문제 유발 가능성 up.
  - 제한된 커스터마이징
    - useAxios는 특정 패턴에 따라 동작 → 모든 요구사항에 충족하지 못할 수도 있음.
      ex) 매우 세밀한 요청 커스터마이징이 필요한 경우, axios의 고급 기능을 사용하는 경우

### useAxios 적절한 사용법

> **[공식문서](https://vueuse.org/integrations/useAxios/)에는 useAxios 사용시 axios와 적절히 섞어가며 쓰는 걸 알 수 있다.**

> **공식문서에 따르면 axios를 통해 인스턴스를 전역으로 먼저 생성하고 해당 인스턴스를 useAxios의 옵션으로 사용할 수 있다.**

useAxios만 활용한다면 아래와 같이,

```tsx
import { useAxios } from '@vueuse/integrations/useAxios';

const { data, isFinished } = useAxios('/api/posts');
```

axios를 통해 전역 인스턴스를 생성한 후, 해당 인스턴스를 useAxios의 옵션으로 활용하는 방법은 아래와 같이 쓸 수 있다.

```tsx
import axios from 'axios';
import { useAxios } from '@vueuse/integrations/useAxios';

const instance = axios.create({
  baseURL: '/api',
});

const { data, isFinished } = useAxios('/posts', { method: 'POST' }, instance);
```

> 현재 우리 프로젝트의 추후 확장성과 유연성을 생각하면 axios와 useAxios를 같이 사용하는 방향이 더 낫다고 판단됨.

---

#### +) Advanced:

[[Advanced] vue에서 axios만 사용하는게 더 편리하지 않을까?](https://www.notion.so/Advanced-vue-axios-ef3f4f36138a4fb0aa5915e76ba2846d?pvs=21)

#### +) 실제 실습 내용:

[[Vue] Vue에서 BE의 API 연결하기 ](https://www.notion.so/Vue-Vue-BE-API-cd6d147f7d4b48298b50cf5354b50420?pvs=21)
