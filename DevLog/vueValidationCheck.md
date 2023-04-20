# 유효성 체크

## Vue에서 input 값 유효성 검사하기

> Quasar라는 프레임워크 기반으로 input, button등의 컴포넌트들을 커스텀했다.
> 

### 코드

> 이메일에 해당하는 정규표현식을 통해 검증 처리하는 메서드를 :rules라는 q-input을 통해 커스텀한 input 태그의 옵션값에 다음과 같이 적용시켜 주었다.
> 

```html
<!-- HTML 코드 부분 -->
<h-input
  class="trial-email-tab__form-input"
  label="Company Email＊"
  v-model="email"
  :rules="emailRules"
/>
```

```tsx
// script 코드 부분
const email = ref('');

const emailRegex = /^[a-z0-9.\-_]+@([a-z0-9-]+\.)+[a-z]{2,6}$/;

const emailRules = [(val: any) => emailRegex.test(val) || 'Please write as right email format'];
```
### 결과 화면

> 정규표현식에 맞는 이메일 형식에 따르지 않는 경우 다음과 같은 css 처리가 자동으로 된다.
> 

![image](https://user-images.githubusercontent.com/73332608/232436986-9a9d26d9-d47b-4f31-afe0-ac74d564bd1e.png)

### Reference

[Documentation | Quasar Framework](https://quasar.dev/docs)

[Email validation using quasar itself](https://forum.quasar-framework.org/topic/5062/email-validation-using-quasar-itself)

## 추가 정리

### validation이 되지 않을 경우 다음 페이지로 넘어가지 않게 하기

> 위에서 validation을 하긴 했지만, 육안으로 틀린 걸 알 수 있는 용도일 뿐 다음 페이지로 넘어갈 때 예외처리를 해주지 않았기 때문에 이 부분도 추가해보았다.
> 

### 코드

```html
<h-btn
  class="trial-email-tab__form-btn"
  color="primary"
  label="NEXT"
  @click="sendEmail"
  :to="{ name: 'SettingDomain' }"
  :disable="!isValidEmail"
/>
```

```tsx
const isValidEmail = computed(() => emailRegex.test(email.value));
```

💡 **위와 같이 isValidEmail을 통해 email이 정해진 정규표현식에 맞지 않으면 버튼이 활성화 되지 못하도록 막아두었다.**