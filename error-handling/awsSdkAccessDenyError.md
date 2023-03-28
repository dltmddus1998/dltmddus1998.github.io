# aws-sdk access deny 에러

## 에러 
![스크린샷 2023-03-28 오후 2 02 04](https://user-images.githubusercontent.com/73332608/228135668-44e85db1-a369-42fa-a2ed-64f6bf298053.png)

Error: User: arn:aws:iam::xxxx:user/xxxx is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::xxxx:user/xxxxx

## 에러 원인
해당 에러는 해당 IAM 유저가 STS AssumeRole을 실행할 수 있는 권한이 없어서 생겼다.

## 해결방안
문제를 해결하려면, 해당 사용자에 연결된 IAM 정책을 확인하고, 해당 arn의 리소스에서 지정된 역할을 가정할 수 있는 필요한 권한이 있는지 확인해야 한다. 또한 사용자가 역할을 가정할 수 있도록 신뢰 정책을 확인해야 할 수도 있다.

해결 방안으로는 aws console에 로그인하여 IAM 서비스에서 직접 해당 정책을 추가하는 방법이 있다. 

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole",
            ],
            "Resource": "*"
        }
    ]
}
```

## 그러나 여전히 같은 에러
😩 **그런데, 이렇게 했는데도 저 에러가 사라지지 않는다…**

🧐 **무엇이 문제일지 리서치해본 결과…**

<aside>
⚡ 이부분은 해결할수 없을 것 같다.

</aside>

→ 왜냐하면 지금 해당 계정에 AdministratorAccess라고 관리자 계정으로서의 Permission Policy를 추가해줬는데도 에러가 없어지지 않기때문에 사용할 수 없는 라이브러리 인 것으로 판단했고 다른 방법을 찾아봐야 할 것 같다.
