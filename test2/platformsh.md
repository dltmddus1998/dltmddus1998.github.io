# Bepol 프로젝트 배포 Dev-log (ing)

## 배포중인 프로젝트
[👆 GITHUB LINK](https://github.com/oxopolitics-internship-for-codestates/BePol)

## 배포 상황

![image](https://user-images.githubusercontent.com/73332608/185758877-c6ad19f2-666f-4e0c-95cd-a3d25bfada71.png)

![image](https://user-images.githubusercontent.com/73332608/185758880-77e975b1-8585-4415-914a-240ddcfc6a90.png)

![image](https://user-images.githubusercontent.com/73332608/185758887-842f9a6a-f5ba-4053-a62e-06a49ffdf2ba.png)

platform.sh를 통해 node.js + mongoDB 프로젝트를 배포하려 하는데 고통스럽다… 에러가 이만저만이 아니다.

1. ssh 중복으로 인한 에러로 2시간 날림… 결국 위처럼 하니 해결 돼버리는 magic…🧙🏻‍♀️
2. 🤮 현재 진행형… 배포는 성공했다…근데 문제가 생겼다. 로그 확인차 platform log를 실행했는데 mongoDB에서 에러가 났다.
    1. parse 관련 에러인데 구글링을 아무리해도 관련 자료는 거의 없고,
    2. 하다못해 node_modules에 들어가서 해당 파일들 분석해봤는데도 답이 없다.
    3. 같이 백엔드 담당했던 팀원에게 SOS를 요청한 상태…

### 🤔 팀원 회의 결과
platform.sh는 자료도 턱없이 부족하고 오류에 대해 구글링해도 나오는게 없기 때문에 더이상 시간을 할애할 이유가 없어 보였다.
지금까지 해놓은게 아깝긴하지만 시간 낭비를 줄이기 위해 익숙한 AWS로 옮겼다.

### 👩🏻‍💻 그래서 배포 결과는 ???
정말 억울하게도 30분도 안돼서 성공했다^^ 

RDS로 데이터베이스를 배포하면 과금이 될 수도 있어서 걱정했는데 MONGODB는 애초에 Atlas로 설정해주면 따로 배포할 필요가 없는 게 다행인 점이었다.
그래서 AWS EC2를 통해 서버 배포를 진행했고 문제없이 배포를 완료했다... 그동안 platform.sh (이전에 Heroku로도 하루정도 허비했다^^)로 날린 시간이 무색하게도 ㅎㅎ

![image](image.png)

### 🤮 그런데 또 문제가? 다행히 바로 해결!
이제 클라이언트는 뭐 별거 없겠지 생각하고 야심차게 Netlify로 간편하게 빌드 및 배포까지 마쳤는데!! 아니 styled-componenets가 적용이 안된다. 🤬

우선 팀원과 이유를 찾아본 결과 styled-components를 글로벌로 설정했을 경우 빌드할 때 스타일을 인식하지 못할 수도 있다고 한다. 

그리고 또 다른 결정적인 문제는 Netlify는 https 통신만 가능하다는 것... 따로 https 설정을 해주지 않아서 결국 돌고 돌아 AWS로 다시 돌아왔다...

<br>

#### 💡 AWS S3 Bucket을 이용한 배포
이건 어렵지 않게 배포했다. styled-component 관련 문제는 위와 같은 경우라 같은 방식으로 해결하니 스타일을 잘 인식했다.

버킷 정책은 아래와 같이 설정해주었다.

그래서 다음과 같이 클라이언트 배포까지 완료했다!

