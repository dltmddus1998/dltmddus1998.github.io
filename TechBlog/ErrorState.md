# Error State vs. Error Exception

---

## 👀 When we use Error State?

> **예측 가능한 에러일 경우 try/catch를 통해 throw문을 남용하지 말고, Error State을 이용하여 에러 표시를 해줄 수 있다.**
> 

```jsx
// error state 사용 예시
type SuccessState = { 
        result: 'success'
    }
    type NetworkErrorState = {
        result: 'fail',
        reason: 'offline' | 'down' | 'timeout'
    }

    type ResultState = SuccessState | NetworkErrorState;
    class NetworkClient {
        tryConnect(): ResultState {
            return {
                result: 'success'
            }
        }
    }
```
