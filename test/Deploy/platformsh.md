![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5967529f-8010-422d-bd7d-eb5f8a4b46a5/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a7c62bc-fea7-4c71-8926-05d81673d678/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8ba61edb-bf20-4ba2-ac99-8566d9a5c865/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ba6b91bf-5ff6-4228-98f1-2af1dc98f30f/Untitled.png)

platform.sh를 통해 node.js + mongoDB 프로젝트를 배포하려 하는데 고통스럽다… 에러가 이만저만이 아니다.

1. ssh 중복으로 인한 에러로 2시간 날림… 결국 위처럼 하니 해결 돼버리는 magic…🧙🏻‍♀️
2. 🤮 현재 진행형… 배포는 성공했다…근데 문제가 생겼다. 로그 확인차 platform log를 실행했는데 mongoDB에서 에러가 났다.
    1. parse 관련 에러인데 구글링을 아무리해도 관련 자료는 거의 없고,
    2. 하다못해 node_modules에 들어가서 해당 파일들 분석해봤는데도 답이 없다.
    3. 같이 백엔드 담당했던 팀원에게 SOS를 요청한 상태…