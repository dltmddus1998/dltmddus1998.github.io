# 프론트에서 OTP 인증 (2차 인증) 구현하기 (ing)

## 개발 이유

> 왤케 안되니…🤦🏼‍♀️
> 

Vue3 Script setup 버전에서 OTP 인증을 구현해보겠다.

업무 진행 중에 필요한 과정인데, 처음엔 Google Authenticator의 오픈소스를 통해 구현하려 했지만 제대로 된 오픈 소스나 open API 관련 정보는 아무리 찾아도 없을뿐더러 있더라도 예전에 작성해놓은 코드라 호환이 제대로 되지 않아 이는 포기했다… 🥹 

## 개발 과정

### 📚 Story

> 우선, vue3 otp open api 이런식으로 구글링해보니 npm library인 **speakeasy**와 **qrcode**를 활용하라는 말이 가장 많았고 npm 공식 사이트에서도 두 라이브러리 모두 사용률이 높았기 때문에 이를 활용하면 쉽게 구현할 수 있을 거라고 생각해서 바로 구현에 들어갔다. (이건 나의 커다란 착각이었다!! 😖)
> 

> 호기롭게 `speakeasy`와 `qrcode`를 다운받은 후 (우리 팀은 타입스크립트를 활용하기 때문에 `@types/speakeasy`와 `@types/qrcode`를 각각 추가 다운로드하였다.) 구현을 진행했는데 갑자기 다음과 같은 에러가 떴다. *‘Error: util.js process is not defined’* 알고보니 otp 라이브러리에 *process.env*를 쓰는 코드가 있었는데 현재 우리 팀의 버전에서는 *import.meta.env* 형식으로 *env*값을 가져오기 때문에 다음과 같은 에러가 뜨는걸로 보였다. 어떻게든 해결하는 방법은 있었겠지만, 거기에 시간을 버리기엔 너무 아까워서 바로 chat GPT에게 speakeasy를 대체할만한 라이브러리에 대해서 물어보았다. speakeasy는 qrcode를 생성하기 위한 url을 생성하는 라이브러리였기 때문에 다행히 어렵지 않게 대체 라이브러리를 찾을 수 있었다.
> 

> 바로 `**@otplib/preset-default**`라는 라이브러리였는데, `speakeasy`에 비해 사용률은 낮지만 꽤 많은 사람들이 찾아쓰는 라이브러리이다.
> 

> 위 과정을 거쳐서 어찌어찌 코드를 다 짠 후 실행을 시켰는데 계속 qrcode를 통해 얻은 토큰이 verify 함수를 거치지 못했다. 계속 invalid한 값이라고 나온다.
> 

> 한시간 가량 서치해본 결과 코드가 제대로 작동하지 않은 두 가지 이유가 있었는데, 먼저 예외처리를 제대로 안했기 때문에 에러의 직접적인 원인을 알 수 없었고 그랬기 때문에 buffer 관련 처리가 되지 않아 에러가 났다는 것도 모를 수 밖에 없었다. (즉, 예외처리만 제대로 했었어도 금방 끝냈을 일이라는 뜻…)


위 과정들을 거치고 바로 아래 코드가 나왔다. (아직은 제대로 구현되지 않았지만 어느정도 구색은 갖췄다.)

```html
<!-- quasar활용 (q-img) -->
<q-img
  class="otp-tab-desc__qr"
  id="qrImg"
  :src="qrCodeImg"
  :to="{ name: 'SettingPassword' }"
/>
```

```tsx
const isValidBase32 = (str: string): boolean => {
  const regex = /^[A-Z2-7]+=*$/;
  return regex.test(str);
};

const base32Decode = (str: string) => {
  const decoded = decode(str);
  if (!isValidBase32(str)) {
    throw new Error('Invalid base32 characters');
  }
  try {
    return buffer.Buffer.from(decoded);
  } catch (error) {
    console.error(`error = `, error);
    throw new Error('Invalid base32 characters');
  }
};

onMounted(() => {
  QRCode.toDataURL(
    otplib.authenticator.keyuri(registStore.email, registStore.username, secret.value),
    (error, url) => {
      if (error) {
        console.error(`error = `, error);
      } else {
        qrCodeImg.value = url;
      }
    }
  );
});

const authenticate = () => {
  const decodedSecret = base32Decode(secret.value);
  console.log(new TextDecoder().decode(decodedSecret));
  try {
    const isValid = otplib.authenticator.verify({
      token: authenticatorCode.value,
      secret: new TextDecoder().decode(decodedSecret),
    });
    console.log(isValid);
    if (isValid) {
      alert('say yeah!!!');
    } else {
      throw new Error('Invalid Decoded Secretdsafjskldf');
    }
  } catch (error) {
    console.error(`error = `, error);
  }
}
```

## Reference

[chat GPT](https://chat.openai.com/)

## 오늘의 결론

‼️ **예외 처리를 습관화하자.**
