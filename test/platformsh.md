![image](https://user-images.githubusercontent.com/73332608/185758877-c6ad19f2-666f-4e0c-95cd-a3d25bfada71.png)

![image](https://user-images.githubusercontent.com/73332608/185758880-77e975b1-8585-4415-914a-240ddcfc6a90.png)

![image](https://user-images.githubusercontent.com/73332608/185758887-842f9a6a-f5ba-4053-a62e-06a49ffdf2ba.png)

platform.sh를 통해 node.js + mongoDB 프로젝트를 배포하려 하는데 고통스럽다… 에러가 이만저만이 아니다.

1. ssh 중복으로 인한 에러로 2시간 날림… 결국 위처럼 하니 해결 돼버리는 magic…🧙🏻‍♀️
2. 🤮 현재 진행형… 배포는 성공했다…근데 문제가 생겼다. 로그 확인차 platform log를 실행했는데 mongoDB에서 에러가 났다.
    1. parse 관련 에러인데 구글링을 아무리해도 관련 자료는 거의 없고,
    2. 하다못해 node_modules에 들어가서 해당 파일들 분석해봤는데도 답이 없다.
    3. 같이 백엔드 담당했던 팀원에게 SOS를 요청한 상태…
