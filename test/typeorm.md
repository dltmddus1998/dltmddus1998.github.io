# TypeORM이란? (ing)

# TypeORM 사용해보기 

## TypeORM으로 데이터베이스 연결하기

> NestJS 실습용 코드 - 회원가입 서버 구현 중…
> 

```tsx
...
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
        ...
    TypeOrmModule.forRoot({ // 1
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'test',
      database: 'test',
      entities: [__dirname + '/**/*.entity{.ts,.js}'], // 2
      synchronize: true, // 3
    }),
  ],
})
export class AppModule {}
```

- AppModule에 TypeOrmModule을 동적 모듈로 가져온다. (1)
- 소스 코드 내에서 TypeORM이 구동될 때 인식하도록 할 엔티티 클래스의 경로를 지정한다. (2)
- synchronize 옵션은 서비스 구동 시 소스 코드 기반으로 데이터베이스 스키마를 동기화할지 여부이다. (3)
    - 로컬 환경에서 구동시 개발의 편의를 위해 true로 한다.

<aside>
⚠️ synchronize 옵션의 경우 true로 하면 서비스 재실행시 데이터베이스가 초기화되므로 production 단계에서는 사용해선 안된다!

</aside>

다음은, TypeOrmModule.forRoot 함수에 전달하는 TypeOrmModuleOptions 객체이다.

```tsx
export declare type TypeOrmModuleOptions = {
    retryAttempts?: number; // 연결시 재시도 회수 (default: 10)
    retryDelay?: number; // 재시도 간의 지연시간 (단위: ms, default: 3000)
    toRetry?: (err: any) => boolean; // 에러가 났을 때 연결을 시도할지 판단하는 함수
    autoLoadEntities?: boolean; // 엔티티를 자동 로드할지 여부
    keepConnectionAlive?: boolean; // 애플리케이션 종료 후 연결을 유지할 지 여부
    verboseRetryLog?: boolean; // 연결 재시도시, verbose 에러메시지를 보여줄지 여부
} & Partial<ConnectionOptions>;
```

### ormconfig.json 활용하기

> Nest는 데이터베이스를 연결하는 또 다른 방법을 제공한다.
루트 디렉토리에 `ormconfig.json` 파일이 있다면 `TypeOrmModule.forRoot()` 에 옵션 객체를 전달하지 않아도 된다.
> 

<aside>
⚠️ JSON 파일에는 엔티티의 경로를 `__dirname` 으로 불러올 수 없으므로 빌드 후 생성되는 디렉토리 이름인 dist를 붙여줘야 한다.

</aside>

```tsx
{
  "type": "mysql",
  "host": "localhost",
  "port": 3306,
  "username": "root",
  "password": "test",
  "database": "test",
  "entities": ["dist/**/*.entity{.ts,.js}"],
  "synchronize": true
}
```

```tsx
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot(),
  ],
})
export class AppModule {}
```

☑️ ormconfig.json으로 기술된 데이터베이스 옵션에는 비밀번호 같은 민감한 정보가 포함되어 있다. 

☑️ 그런데 JSON은 dotenv로 읽어온 값을 넣을 수 없기 때문에 개발환경에 따라 ormconfig.json파일을 프로비저닝할 때 환경에 맞는 파일로 교체해 주는 장치가 필요하다.

📌 **이 부분은 다른 페이지에 따로 정리하겠다.**

## 요청한 정보 데이터베이스에 저장하기

> 위에서 언급했듯이 회원가입 관련 서버를 실습 중에 typeORM을 진행중이다.
따라서, 회원 가입을 요청한 유저의 정보를 저장해보려 한다.
> 

```tsx
import { Column, Entity, PrimaryColumn } from "typeorm";

@Entity("User")
export class UserEntity {
  @PrimaryColumn()
  id: string;

  @Column({ length: 30 })
  name: string;

  @Column({ length: 60 })
  email: string;

  @Column({ length: 30 })
  password: string;

  @Column({ length: 60 })
  signupVerifyToken: string;
}
```

☑️ 위의 유저 엔터티를 데이터베이스에서 사용할 수 있도록 TypeOrmModuleOptions의 entities 속성의 값으로 넣어주면 된다. (→ 이미 위에서 구현했다.)

```tsx
{
    ...
  "entities": ["dist/**/*.entity{.ts,.js}"],
  ...
}
```