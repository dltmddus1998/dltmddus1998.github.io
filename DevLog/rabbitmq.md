[Documentation | NestJS - A progressive Node.js framework](https://docs.nestjs.com/microservices/rabbitmq#rabbitmq)

***RabbitMQ***는 **분산 시스템에서 메시지 지향 미들웨어를 구현**하는 데 자주 사용되는 메시지 브로커이다.

## 실습

### 다운로드

```
$ brew install rabbitmq
```

```
$ pnpm add amqp-connection-manager amqplib
```

### 코드 구조

```
📦apps
 ┣ 📂api-home
 ┃ ┣ 📂src
 ┃ ┃ ┣ 📂auth
 ┃ ┃ ┃ ┣ 📂guards
 ┃ ┃ ┃ ┃ ┣ 📜auth.guard.ts
 ┃ ┃ ┃ ┃ ┣ 📜google-auth.guard.ts
 ┃ ┃ ┃ ┃ ┣ 📜jwt-auth.guard.ts
 ┃ ┃ ┃ ┃ ┗ 📜naver-auth.guard.ts
 ┃ ┃ ┃ ┣ 📜auth.controller.ts
 ┃ ┃ ┃ ┣ 📜auth.module.ts
 ┃ ┃ ┃ ┗ 📜auth.service.ts
 ┃ ┃ ┣ 📜app.module.ts
 ┃ ┃ ┗ 📜main.ts
 ┃ ┗ 📜tsconfig.app.json
 ┗ 📂hybrix-svc-auth
 ┃ ┣ 📂src
 ┃ ┃ ┣ 📂constants
 ┃ ┃ ┃ ┗ 📜exception.ts
 ┃ ┃ ┣ 📂service
 ┃ ┃ ┃ ┣ 📜auth.service.ts
 ┃ ┃ ┃ ┗ 📜jwt.service.ts
 ┃ ┃ ┣ 📂strategy
 ┃ ┃ ┃ ┗ 📜jwt.strategy.ts
 ┃ ┃ ┣ 📜auth.controller.ts
 ┃ ┃ ┣ 📜auth.module.ts
 ┃ ┃ ┗ 📜main.ts
 ┃ ┗ 📜tsconfig.app.json
```

⚠️ **코드 양이 많아서 RabbitMQ 사용시 어떤 형식으로 코드를 작성하는지만 보여주는 용도로 코드를 보여주겠다.**

### auth-svc 코드

🔐 **auth.service.ts**

```tsx
@Injectable()
export class AuthService {
  constructor(
    @Inject(JwtService)
    private readonly jwtService: JwtService,
    private config: ConfigService,
    @Inject('DATABASE_CONNECTION')
    private db: Db,
  ) {}
	.
	.
	.
	public async google(): Promise<SvcResponse> {
    return {
      statusCode: HttpStatus.OK,
      data: 'success',
    };
  }
	.
	.
	.
}
```

🕹 **auth.controller.ts**

```tsx
import { MessagePattern, Payload } from '@nestjs/microservices';

@Controller()
export class AuthController {
  @Inject(AuthService)
  private readonly service: AuthService;
	.
	.
	.
	@MessagePattern('google')
  private async google(@Payload() payload: LoginRequestDto) {
    return this.service.google();
  }
	.
	.
	.
}
```

🖇 **auth.module.ts**

```tsx
@Module({
  imports: [
    AccountModule,
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: [`${__dirname}/../../.env.${process.env.NODE_ENV}`],
    }),
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        secret: config.get<string>('JWT_SECRET'),
        signOptions: { expiresIn: '365d' },
      }),
    }),
    DatabaseModule,
    RmqModule,
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtService, JwtStrategy],
})
export class AuthModule {}
```

### auth-gateway 코드

🔐 **auth.service.ts**

```tsx
@Injectable()
export class AuthService {
  constructor(
    @Inject('AUTH_SERVICE')
    private authClient: ClientProxy,
  ) {}

  public async validateToken(token: string): Promise<any> {
    const result = firstValueFrom(this.authClient.send('validateToken', token));
    return result;
  }

  public async updateLoginInfo(
    updateLoginInfoRequestDto: UpdateLoginInfoRequestDto,
  ): Promise<any> {}
}
```

🕹 **auth.controller.ts**

```tsx

@Controller('auth')
export class AuthController {
  constructor(
    @Inject('AUTH_SERVICE')
    private authClient: ClientProxy,
    private readonly config: ConfigService,
  ) {}
	.
	.
	.
	@UseGuards(GoogleAuthGuard)
  @Get('/google')
  private async google(query: any): Promise<SvcResponse> {
    return await firstValueFrom(this.authClient.send('google', query));
  }
	.
	.
	.
}
```

🖇 **auth.module.ts**

```tsx
import { RmqModule, RmqService } from '@hybrix/common';
import { Module } from '@nestjs/common';
import { ClientsModule } from '@nestjs/microservices';
import { AuthController } from './auth.controller';
import { AuthService } from './auth.service';

@Module({
  imports: [
    ClientsModule.registerAsync([
      {
        imports: [RmqModule],
        name: 'AUTH_SERVICE',
        inject: [RmqService],
        useFactory: (rmqService: RmqService) =>
          rmqService.getOptions(process.env.RABBIT_MQ_AUTH_QUEUE),
      },
    ]),
  ],
  controllers: [AuthController],
  providers: [AuthService],
})
export class AuthModule {}
```

### 실행 후 RabbitMQ GUI

![image](https://user-images.githubusercontent.com/73332608/225487442-57c95e9a-71d1-4aea-9af0-a4d7d71094e1.png)

## Error-Handling

### ⚠️ Error

<img width="868" alt="error" src="https://user-images.githubusercontent.com/73332608/225487474-937ad445-03f0-49aa-a3c5-eab75c829d35.png">

### 🤔 원인 및 해결

처음에는 의존성 문제다 보니 당연히 코드에 문제가 있는 줄 알고 (모듈 코드에 provider라던지 exports 같은 속성들에 무언가가 빠져있는 경우처럼) 모두 뜯어봤는데 전혀 문제가 없었다.

chatGPT한테 물어보니 코드에 문제가 없을경우 RabbitMQ 서버의 문제일수도 있다고 해서 서버도 껐다 켜보고 GUI도 로그아웃했다가 다시 들어가보고 모든 경우를 다 실행해보았는데도 해결되지 않았다.

근데 다음날 출근해서 똑같이 해보니까 갑자기 된다… 이유를 모르겠지만 어제는 아마 RabbitMQ 서버가 제대로 켜지지 않은 상태에서 계속 코드를 실행시키다보니 에러가 난 것 같다.
