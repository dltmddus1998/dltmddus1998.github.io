# Error State vs. Error Exception

---

## ๐ When we use Error State?

> **์์ธก ๊ฐ๋ฅํ ์๋ฌ์ผ ๊ฒฝ์ฐ try/catch๋ฅผ ํตํด throw๋ฌธ์ ๋จ์ฉํ์ง ๋ง๊ณ , Error State์ ์ด์ฉํ์ฌ ์๋ฌ ํ์๋ฅผ ํด์ค ์ ์๋ค.**
> 

```jsx
// error state ์ฌ์ฉ ์์
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
