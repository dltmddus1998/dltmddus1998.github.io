# Bepol 프로젝트 중 투표 현황 이메일 전송 기능 구현

## 투표 현황 이메일 전송 기능 구현

백엔드 부분 구현이 완료돼서 Advanced로 구현하기로 한 부분을 시작했다. 투표 현황에 대해 작성자에게 이메일로 전송해주는 기능이다.

## 로직

```
1️⃣ 발의문들 중 마감 하루 남은 것들 필터링 (마감 안된걸로)
2️⃣ 해당 발의문의 작성자 이메일 찾기 (User collection)
3️⃣ 해당 발의문 투표 통계 페이지 (client) 캡쳐 기능 - puppeteer 사용
4️⃣ 캡쳐한 화면을 node-mailer에서 attachment 옵션으로 보내기 
5️⃣ 중복메일이 가지 않도록 Post 컬렉션의 sendEmailStatus가 false일 때만 메일가게 설정 (error-handling)
6️⃣ 메일 전송 후 imgs폴더의 사진들은 삭제하기
```

## 기능 구현

> 먼저 마감이 하루 남은 발의문들을 필터링해준다.
> 

```jsx
let thirtyPercentOverPosts = [];
const postsList = await Post.find(
  {},
  {
    _id: 1,
    username: 1,
    title: 1,
    agrees: 1,
    disagrees: 1,
    userId: 1,
    createdAt: 1,
    sendEmailStatus: 1,
  }
);
for (let post of postsList) {
  const { createdAt } = post;
  const date = new Date(createdAt);
  const year = date.getFullYear();
  const month = date.getMonth();
  const day = date.getDate();

  const afterOneMonth = new Date(year, month + 1, day);
  const oneDayBeforeEnd = new Date(year, month + 1, day - 1);

  if (
    oneDayBeforeEnd.toLocaleDateString() ===
      new Date().toLocaleDateString() &&
    afterOneMonth.getTime() > new Date().getTime()
  ) {
    // 마감되지 않은 발의문 중 마감 하루 남은 발의문 리스트 filtering
    thirtyPercentOverPosts.push(post);
  }
}
```

> 이제, 해당 발의문의 작성자를 User 컬렉션에서 찾아서 해당 사용자의 이메일을 통해 기능을 구현해주면 된다.
> 

☑️ node-mailer을 활용하여 이메일 전송 기능을 구현했다. 

<aside>
⚙ ☑️ 먼저 config폴더에서 사전 설정 작업을 진행한다. 
☑️ .env 파일에서 sendEmailUser와 sendEmailPassword를 선언해주는데, 여기서 sendEmailPassword는 구글계정에서 앱 비밀번호를 따로 설정해준 후 해당 비밀번호를 복사해오면 된다.

</aside>

```jsx
import nodemailer from "nodemailer";
import dotenv from "dotenv";
dotenv.config();

const { sendEmailUser, sendEmailPassword } = process.env;

export const transport = nodemailer.createTransport({
  service: "Gmail",
  host: "smtp.gmail.com",
  port: 465,
  secure: true,
  auth: {
    user: sendEmailUser,
    pass: sendEmailPassword,
  },
});
```

☑️ 이후 원래 파일로 돌아와서 옵션들을 설정해준다.

```jsx
const emailOptions = {
  from: sendEmailUser,
  to: email,
  subject: `안녕하세요. BePol입니다.`,
  html: `${username}님이 작성하신 ${title}에 관한 청원 투표 현황입니다.
      <br><br>
      <img src="cid:stats">
    `,
  attachments: [
    {
      filename: "stats.png",
      path: `imgs/stats${_id}.png`,
      cid: "stats",
    },
  ],
};
```

*오류를 최소화 하기 위해 if문으로 경우를 모두 나눠서 진행한다.*

⚠️ imgs폴더에 캡쳐한 클라이언트 화면들을 모두 저장하여 해당 파일을 이메일로 보내는 방식인데, 폴더에 해당 파일이 없는 경우까지 나눠야 에러가 나지 않았다. (그 경우를 나누지 않으면 fs관련 에러가 계속 나서 이렇게 처리해주니 에러가 없어졌다.)

- imgs폴더에 해당 파일이 있으면서 Post 컬렉션의 sendEmailStatus(이메일 전송 여부)가 false 인 경우 1️⃣
- imgs폴더에 해당 파일이 없으면서 Post 컬렉션의 sendEmailStatus가 false인 경우 2️⃣

**경우 1️⃣**

> Post 컬렉션의 이메일 상태 데이터를 업데이트해주는데, 여기서 오류가 나는 경우 메일이 보내지지 않도록 에러 핸들링을 해줬다.
> 

```jsx
Post.updateOne({ _id }, { sendEmailStatus: true })
  .then(async () => {
		// updateOne에 오류가 생기지 않을때만 메일이 보내지도록 처리
    transport.sendMail(emailOptions); 
  })
  .then(() => {
    console.log(
      `Emails are sent in ${new Date().toLocaleDateString()}`
    );
  })
  .catch(async (err) => {
    console.log(err);
  });
```

**경우 2️⃣**

> 에러 핸들링 방식은 위와 동일하다.
> 

☑️ 클라이언트에서 통계 화면에 해당하는 브라우저 전체 화면을 캡쳐하여 파일로 저장하기 위해 puppeteer를 사용했다. 

☑️ imgs폴더에 `stats(해당 발의문의 _id).png` 형식으로 캡쳐 파일을 저장하는 옵션을 설정했다. 

```jsx
puppeteer.launch().then(async (browser) => {
  return browser.newPage().then(async (page) => {
    return page
      .goto("http://localhost:3000/write")
      .then(async () => {
        await page.screenshot({
          fullPage: true, // 전체페이지 캡쳐 옵션
          path: `imgs/stats${_id}.png`, // 캡쳐본 파일명
        });
      })
      .then(() => browser.close())
      .then(async () => {
        await Post.updateOne({ _id }, { sendEmailStatus: true });
      })
      .then(async () => {
        transport.sendMail(emailOptions); // updateOne에 오류가 생기지 않을때만 메일이 보내지도록 처리
      })
      .then(() => {
        console.log(
          `Emails are sent in ${new Date().toLocaleDateString()}`
        );
      })
      .catch(async (err) => {
        console.log(err);
      });
  });
});
```

☑️ 마지막으로 위에 구현했던 부분을 export해 app.js에서 node-cron을 이용하여 주기 설정을 했다. 

☑️ imgs폴더에 파일이 한없이 쌓이는 것을 방지하기 위해 메일이 보내진 후에는 해당 폴더 내부가 모두 삭제되도록 설정했다.

```jsx
// 투표 현황 메일 - 매일 오전 9시
cron.schedule("59 8 1-31 * *", async () => {
  await sendMailStatusRepository.sendMailStats();
});

// 메일 보낸 후 imgs 폴더 비워주기
cron.schedule("59 9 1-31 * *", async () => {
  fsExtra.emptyDirSync("imgs/");
});
```
