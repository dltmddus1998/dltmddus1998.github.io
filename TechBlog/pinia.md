## Pinia를 사용해야 하는 이유?

> vue3에서 새로 나온 상태관리 방식이다. 원래는 ref, reactive 등을 통해 상태관리를 하는게 보편적이긴 하나 다음과 같은 문제가 있다.
> 

Single Page Application (SPA) 방식에 해당되는 경우이지만, 서버측에서 렌더링되는 경우 애플리케이션이 보안 취약성에 노출된다. 

- Devtools 지원
    - 액션, 뮤테이션을 추적할 수 있는 타임라인
    - 사용되는 컴포넌트에서 스토어가 나타납니다.
    - 시간 여행 및 디버깅이 더 쉬워집니다.
- 핫 모듈 교체
    - 페이지를 다시로드하지 않고 스토어를 수정할 수 있습니다.
    - 개발 중에 기존 상태를 유지합니다.
- 플러그인: 플러그인으로 Pinia 기능을 확장합니다.
- JS 사용자를 위한 적절한 TypeScript 지원 또는 자동 완성
- 서버 측 렌더링 지원

## Pinia 사용법

### Define Store

가장 먼저해야할 건, Store를 정의하는 건데 defineStore이라는 사용한다.

```tsx
defineStore('[Store 이름]', { [Options] })
```

이제 Options라고 써진 부분에 다음 세 가지가 들어간다.

- state
    - 초깃값을 저장하는 공간이라고 생각하면 된다.
    
    ```tsx
    export const useUserStore = defineStore('user', {
      state: () => {
        return {
          // for initially empty lists
          userList: [] as UserInfo[],
          // for data that is not yet loaded
          user: null as UserInfo | null,
        }
      },
    })
    
    interface UserInfo {
      name: string
      age: number
    }
    ```
    
- getters
    - getters는 Store의 state를 위한 computed value들의 값이다.
    - 위에서 정의한 state를 첫 번째 파라미터를 받게된다.
    
    ```tsx
    export const useCounterStore = defineStore('counter', {
      state: () => ({
        count: 0,
      }),
      getters: {
        // automatically infers the return type as a number
        doubleCount(state) {
          return state.count * 2
        },
        // the return type **must** be explicitly set
        doublePlusOne(): number {
          // autocompletion and typings for the whole store ✨
          return this.doubleCount + 1
        },
      },
    })
    ```
    
- actions
    - actions는 컴포넌트 내에서 메서드로서 사용할 수 있다.
    
    ```jsx
    import { mande } from 'mande'
    
    const api = mande('/api/users')
    
    export const useUsers = defineStore('users', {
      state: () => ({
        userData: null,
        // ...
      }),
    
      actions: {
        async registerUser(login, password) {
          try {
            this.userData = await api.post({ login, password })
            showTooltip(`Welcome back ${this.userData.name}!`)
          } catch (error) {
            showTooltip(error)
            // let the form component display the error
            return error
          }
        },
      },
    })
    ```
    

## Reference

[Pinia 🍍](https://pinia.vuejs.org/introduction.html)