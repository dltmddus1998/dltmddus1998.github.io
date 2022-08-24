# TypeORM이란? (ing)

## TypeORM 사용해보기 

### TypeORM으로 데이터베이스 연결하기

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

#### ormconfig.json 활용하기

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

### 요청한 정보 데이터베이스에 저장하기

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

이후 서버를 실행시키면 다음과 같이 User 테이블이 생성됐다.

![image](https://user-images.githubusercontent.com/73332608/186462766-3a6e7370-27c0-499a-91bd-e11d9cb851c5.png)

```tsx
...
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserEntity } from './entity/user.entity';

@Module({
  imports: [
        ...
        TypeOrmModule.forFeature([UserEntity]),
    ],
    ...
})
export class UsersModule {}
```

☑️ UsersModule에서 forFeature() 메소드로 유저 모듈 내에서 사용할 저장소를 등록한다.

```tsx
...
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { UserEntity } from './entity/user.entity';

export class UsersService {
  constructor(
        ...
    @InjectRepository(UserEntity) private usersRepository: Repository<UserEntity>,
  ) { }
    ...
}
```

☑️ UsersService에서 `@InjectRepository()` 데코레이터로 유저 레포지토리를 주입한다.

☑️ 여기서 주입한 레포지터리를 통해 다음과 같이 데이터베이스에 저장한다.

```tsx
...
import { ulid } from "ulid";

...
private async saveUser(name: string, email: string, password: string, signupVerifyToken: string) {
  const user = new UserEntity();
  user.id = ulid();
  user.name = name;
  user.email = email;
  user.password = password;
  user.signupVerifyToken = signupVerifyToken;
  await this.usersRepository.save(user);
}
...
```

☑️ 다음은 기존 유저 정보를 확인하는 함수이다.

☑️ 가입시 이미 존재하는 회원이면 422 에러를 내도록 한다. (`UnprocessableEntityException`)

```tsx
async createUser(name: string, email: string, password: string) {
  const userExist = await this.checkUserExists(email);
  if (userExist) {
    throw new UnprocessableEntityException("해당 이메일로는 가입할 수 없습니다."); // 422 error 
  }
	...
}

...

private async checkUserExists(emailAddress: string): Promise<boolean> {
  const user = await this.userRepository.findOne({
    where: {
      email: emailAddress,
    },
  });

  return user !== undefined;
}
```

### 트랜잭션 처리

<aside>
💡  트랜잭션은 요청을 처리하는 과정에서 데이터베이스에 변경이 일어나는 요청을 독립적으로 분리하고 에러가 발생했을 경우 이전 상태로 되돌리게 하기 위해 데이터베이스에서 제공하는 기능이다.

</aside>

📌 TypeORM에서 트랜잭션을 사용하는 방법은 아래와 같이 3가지가 있다.

- `QueryRunner`를 이용하여 단일 DB 커넥션 상태를 생성하고 관리하기
- `transaction` 객체를 생성해서 이용하기
- `@Transaction`, `@TransactionManager`, `@TransactionRepository` 데코레이터를 사용하기.

⚠️ 이중 데코레이터를 사용하는 방식은 Nest에서 권장하지 않는다. 마지막 방법은 제외하자.

#### QueryRunner 클래스를 사용하는 방법

> QueryRunner를 사용하면 트랜잭션을 완전히 제어할 수 있다.
> 

```tsx
...
import { Connection, ... } from 'typeorm';

@Injectable()
export class UsersService {
  constructor(
        ...
        private connection: Connection,
  ) { }
    ...
}
```

☑️ 먼저 typeorm에서 제공하는 Connection 객체를 주입했다.

👩🏻‍💻 **참고 문서대로 하던 중 다음과 같은 에러를 만났다.**

![image](https://user-images.githubusercontent.com/73332608/186462871-d0900669-a0d0-49c2-855b-4459ed03b632.png)

👉 **구글링을 해보니, 0.3.0 버전 업데이트 당시 Connection이 DataSource라는 이름으로 변경됐다고 나온다.(지금 사용중인 typeorm version: ^0.3.7)**

![image](https://user-images.githubusercontent.com/73332608/186463023-1081f257-0622-4319-b1b6-880577f3a35d.png))

👉 복잡할 건 없다. 위 코드를 다음과 같이 수정해주기만 하면 된다. (변수명은 고칠 필요는 없는데 가독성을 위해…)

```tsx
...
import { DataSource, ... } from 'typeorm';

@Injectable()
export class UsersService {
  constructor(
        ...
        private dataSource: DataSource,
  ) { }
    ...
}
```

☑️ DataSource 객체에서 트랜잭션 생성이 가능하다.

☑️ 유저를 저장하는 로직에 트랜잭션을 걸어보자. (위에서 구성해준 saveUser를 지우고 대신 saveUserUsingQueryRunner을 구성해보자)

```tsx
private async saveUserUsingQueryRunner(
    name: string,
    email: string,
    password: string,
    signupVerifyToken: string
  ) {
    const queryRunner = this.dataSource.createQueryRunner();

    await queryRunner.connect();
    await queryRunner.startTransaction(); // 트랜잭션 시작

    try {
      const user = new UserEntity();
      user.id = ulid();
      user.name = name;
      user.email = email;
      user.password = password;
      user.signupVerifyToken = signupVerifyToken;

      await queryRunner.manager.save(user);

      await queryRunner.commitTransaction(); // 에러 x -> commit
    } catch (e) {
      await queryRunner.rollbackTransaction(); // 에러 o -> rollback
    } finally {
      await queryRunner.release(); // QueryRunner 해제
    }
  }
```

#### transaction 객체를 생성해서 사용하는 방법

💡 또 다른 방법으로 dataSource 객체 내의 transaction 메서드를 바로 이용하는 방법도 있다. 

👉 해당 메서드의 주석을 보면 다음과 같다.

> **transaction 메소드는 주어진 함수 실행을 트랜잭션으로 래핑한다.**
> 

> **모든 데이터베이스 연산은 제공된 엔티티 매니저를 이용하여 실행해야 한다.**
> 

☑️ transaction 메서드는 `EntityManager`를 콜백으로 받아 사용자가 어떤 작업을 수행할 함수를 작성할 수 있도록 해준다.

```tsx
private async saveUserUsingTransaction(
    name: string,
    email: string,
    password: string,
    signupVerifyToken: string
  ) {
    await this.dataSource.transaction(async (manager) => {
      const user = new UserEntity();
      user.id = ulid();
      user.name = name;
      user.email = email;
      user.password = password;
      user.signupVerifyToken = signupVerifyToken;

      await manager.save(user);
    });
  }
```
