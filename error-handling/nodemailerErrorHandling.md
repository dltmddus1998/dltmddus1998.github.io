# 비밀번호 찾기 서버 구현 에러 - nodemailer 관련

## **어떤 에러인가요?**

- 이메일을 이용한 비밀번호 찾기 서버 구현 중 발생한 에러

## **에러 메시지**

`Error: Invalid login: 535-5.7.8 Username and Password not accepted. Learn more at`

## **에러 핸들링 방법**

이메일을 송신할 해당 gmail 주소에서 다음과 같은 사전 작업이 필요하다.

1. `보안 수준이 낮은 앱` 엑세스 허용 하기
    1. 여기서 나온 코드를 이메일 전송에 필요한 `transport`를 선언 및 할당 할 때 이메일을 송신할 메일주소의 비밀번호에 쓰면 된다. (`.env` 사용하기)
2. 해당 Google 계정에 대한 엑세스 허용하기

```jsx
const transporter = nodemailer.createTransport({
            service: 'Gmail',
            port: ---,
            secure: true,
            auth: {
                user: '------------',
                pass: '-------------',
            },
        });
```
