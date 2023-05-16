# AWS EC2 생성 및 기본 설정

## Launch Instance (EC2 생성)

1. [aws.amazon.com](http://aws.amazon.com) 에 로그인한 후 ec2 인스턴스를 생성한다.
2. 원하는 호스트 명을 생성한다.
3. Application and OS Images를 선택한다. (주로 리눅스 사용)
4. Instance Type을 선택한다 (Free-tier로는 t2.micro만 가능 → 용량이 클 경우 이를 업데이트 해야한다.)
5. Key pari를 선택한다. (계속 새로 만들지 말고 기존의 key를 잘 보관해두고 사용하는 게 편하다.)
6. ‼️ **Network setting**
    
    ✓ 미리 생성해놓은 VPC가 있으면 Select existing security group을 선택한다.
    
    단, 이를 선택하면 public IP가 자동으로 배정되지 않기 때문에 **Edit**을 통해 따로 설정해주어야 한다.
    
    → Subnet에서 원하는 subnet을 선택하고, Auto-assign public IP을 enable로 설정한다.
    
    → Advanced network configuration 토글을 열면 Primary IP를 지정할 수 있다. 
    
7. 이 후 Launch Instance를 누르면 기본적으로 작동하는 ec2를 생성할 수 있다.

👉🏻 **이 후 ssh 연결을 통해 해당 서버에 접속을 하면 다음과 같이 기본적으로 필요한 라이브러리들을 설치하면 된다.**

## 기본 라이브러리 설치

1. nvm 설치
    
    ```
    > curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
    
    > source ~/.nvm/nvm.sh
    
    > nvm install node
    ```
    
2. git 설치
    
    ```
    > yum install -y git
    ```
    
3. docker 설치
    
    ```
    > yum install docker
    
    > systemctl start docker
    
    > systemctl enable docker 
    
    > docker run -d --name rabbitmq -p 5672:5672 -p 8080:15672 --restart=unless-stopped rabbitmq
    ```
    
4. build 및 pm2 실행
    
    ```
    > pnpm run build:all
    
    > pm2 start pnpm --name "[pm2 이름 - 디렉토리명]" -- start:prod(package.json에서 미리 설정해놓은 명령어) [디렉토리명]
    (build:all, start:prod는 package.json에서 생성한 명령어이므로 다르게 적용된다.)
    ```