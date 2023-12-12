# npm 설치 권한 문제

## What happend?

> 평소처럼 npm install로 패키지 설치를 하려고 했는데 계속 아래와 같은 에러가 떴다.
> 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/793761ee-1da2-4c3c-8209-ce8f1147ec2d/267cf7c3-2390-430c-afd2-c0d6a5c7da9b/Untitled.png)

서치해보니, npm을 사용하여 패키지를 설치하려고 할 때 권한 문제가 발생한 것이었다.

에러 메시지를 보면 “npm 캐시 폴더 안에 일부 파일이 루트 사용자에게 속해 있다” 라고 나와있다.

이런 경우에는, npm 설치에 문제가 생길 수 있는데 이를 해결하려면 다음과 같은 명령어를 실행하라고 한다.

`**sudo chown -R 510:20 “<<사용자 디렉토리>>/.npm”**`

위 명령어는 .npm 폴더와 그 안의 파일들의 소유자를 나의 사용자와 그룹으로 변경해준다. 

이 후 npm install을 실행하면 패키지 설치가 문제없이 진행되는 것을 알 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/793761ee-1da2-4c3c-8209-ce8f1147ec2d/1bccc3e8-50b8-49d3-9a99-94f070b6fa27/Untitled.png)