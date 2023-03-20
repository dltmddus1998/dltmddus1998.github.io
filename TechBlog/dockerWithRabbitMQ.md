# ⬇️ Docker CLI 다운로드

```
$ brew install docker docker-compose docker-machine
```

# Docker에서 RabbitMQ 생성하기

```
$ docker run -d --name rabbitmq_hybrix -p 5672:5672 -p 8080:15672 --restart=unless-stopped rabbitmq:management
```

![image](https://user-images.githubusercontent.com/73332608/226245859-e39412f4-487e-4c98-93f1-4300cd5efd7e.png)

✐ **위와 같이 Docker에 RabbitMQ 서버가 생성됨을 알 수 있다.**

**✐ 이후 내가 작성한 백엔드 코드를 (nestjs) 실행시킨 후 RabbitMQ GUI로 접속해보니 아래와 같이 내가 작성한 auth관련 queue가 제대로 실행됐음을 알 수 있다.**

![image](https://user-images.githubusercontent.com/73332608/226245874-679ae40f-c9ee-48ae-bada-8a56e5185c8b.png)
