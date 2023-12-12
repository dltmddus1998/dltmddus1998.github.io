# npm 설치 권한 문제

## What happend?

> 평소처럼 npm install로 패키지 설치를 하려고 했는데 계속 아래와 같은 에러가 떴다.
> 

![image](https://github.com/dltmddus1998/dltmddus1998.github.io/assets/73332608/c2a2faa6-bebd-4050-b9de-2ab3df6a77bc)


서치해보니, npm을 사용하여 패키지를 설치하려고 할 때 권한 문제가 발생한 것이었다.

에러 메시지를 보면 “npm 캐시 폴더 안에 일부 파일이 루트 사용자에게 속해 있다” 라고 나와있다.

이런 경우에는, npm 설치에 문제가 생길 수 있는데 이를 해결하려면 다음과 같은 명령어를 실행하라고 한다.

`**sudo chown -R 510:20 “<<사용자 디렉토리>>/.npm”**`

위 명령어는 .npm 폴더와 그 안의 파일들의 소유자를 나의 사용자와 그룹으로 변경해준다. 

이 후 npm install을 실행하면 패키지 설치가 문제없이 진행되는 것을 알 수 있다.

![image](https://github.com/dltmddus1998/dltmddus1998.github.io/assets/73332608/19606e27-724e-4fa2-a750-16583b0cbcd2)
