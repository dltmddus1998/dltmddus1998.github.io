# 유효성 체크

## Vue에서 input 값 유효성 검사하기

> Quasar라는 프레임워크 기반으로 input, button등의 컴포넌트들을 커스텀했다.
> 

### 코드

> 이메일에 해당하는 정규표현식을 통해 검증 처리하는 메서드를 :rules라는 q-input을 통해 커스텀한 input 태그의 옵션값에 다음과 같이 적용시켜 주었다.
> 

```html
<h-input
  class="trial-email-tab__form-email"
  label="Work Email＊ (editable)"
  v-model="email"
  :rules="[(val) => !!val || 'Email is missing', validateEmail]"
  type="email"
/>
```

```tsx
<script setup lang="ts">
.
.
.
const validateEmail = (email: string) => {
  const regex = /^[a-z0-9.\-_]+@([a-z0-9-]+\.)+[a-z]{2,6}$/;
  return regex.test(email) || 'Invalid Email';
};
.
.
.
</script>
```

### 결과 화면

> 정규표현식에 맞는 이메일 형식에 따르지 않는 경우 다음과 같은 css 처리가 자동으로 된다.
> 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/270a9197-503b-4e09-ab13-f27566d686e0/Untitled.png)

### Reference

[Documentation | Quasar Framework](https://quasar.dev/docs)

[Email validation using quasar itself](https://forum.quasar-framework.org/topic/5062/email-validation-using-quasar-itself)