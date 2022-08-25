# GraphQL의 개념 정리 및 실습

# 💡 기존 API 호출 방식의 한계

클라이언트와 서버 통신시 보통 서버가 구현해놓은 API를 호출해 데이터를 보내거나 받아온다. 보통 앱에서 하나의 View를 그리기 위해서는 여러 번 API를 호출해야하며, 호출을 통해 받아온 데이터를 조합해 사용해야 한다. 

예를 들어 여러번 REST API를 호출해 하나의 View를 만들어내는 경우를 생각해보면, 앱의 페이지가 복잡해질 수록 많은 호출을 해야하고 데이터 조합을 위해 순차적인 처리가 들어가야 하는 경우가 많아지기 때문에 데이터를 조합하는 것은 매우 복잡해진다.

이를 편하게 하기 위해 데이터 흐름을 만들고 해당 흐름에 순차 처리 로직을 위한 로직을 넣는 방식의 프로그래밍이 많이 사용되었다. 대표적인 것으로 Rx와 Coroutine의 Flow 등이 있다. 하지만 flatmap이나 map 등으로 데이터를 변환시키는 것은 결국 우리가 하나의 페이지를 그리기 위한 데이터를 조합하는 것을 클라이언트 단에서 처리해주어야 하므로 클라이언트 단의 로직이 복잡해지는 문제가 있다. 로직이 복잡해지는 것은 유지보수가 어려워진다는 뜻이다.

기존의 REST API나 다른 API 호출 방식은 이 문제를 해결할 수 없다. 기존 API들에서 사용자는 서버에서 정의한 데이터 구조만을 한 번에 하나씩 가져올 수 있기 때문이다. 이를 해결하려면 앱 단에서 직접 뷰에 보여질 쿼리를 만들어 모든 데이터를 한 번에 가져와야 한다.

---

# 🧬 GraphQL의 문제점 해결

> **GraphQL은 API의 쿼리 언어이며, 데이터에 대해 정의한 타입 시스템을 사용하는 실행중인 쿼리들을 위한 server-side 런타임이다.**
> 

<aside>

✅ GraphQL은 클라이언트에서 필요한 데이터만을 쿼리할 수 있도록 해서 위에서 언급한 문제들을 매우 직관적이고 깔끔하게 해결한다.

</aside>

☑️ GraphQL은 어느 특정 데이터베이스나 저장 엔진에 연결되어 있지 않고 대신 기존 코드와 데이터로 뒷받침된다.

☑️ GraphQL 서비스는 해당 타입과 필드를 정의한 후 다음 각 타입의 각 필드에 대한 기능을 제공하여 생성된다.

☑️ 예를 들어,  로그인한 사용자(me)와 해당 사용자의 이름을 알려주는 GraphQL 서비스는 다음과 같다.

```graphql
type Query {
	me: User
}

type User {
	id: ID
	name: String
}
```

☑️ 각 유형의 각 필드에 대한 기능과 함께인 경우

```jsx
function Query_me(request) {
	return request.auth.user;
}

function User_name(user) {
	return user.getName();
}
```

☑️ GraphQL 서비스가 실행된 후 (일반적으로 웹 서비스의 URL에서) GraphQL 쿼리를 수신하여 유효성을 검사하고 실행할 수 있다. 

☑️ 서비스가 먼저 쿼리가 정의된 타입과 필드만 참조하는지 확인한 후 제공된 함수를 실행하여 결과를 생성한다.

```graphql
{
	me {
		name
	}
}
```

☑️ 예를 들어 위 쿼리는 아래의 JSON 결과값을 생산할 수 있다.

```json
{
	"me" {
		"name": "Luke Skywalker"
	}
}
```

---

# 🧐 GraphQL 서버 쿼리하기

## 필드 (Fields)

> **GraphQL은 객체의 특정 필드를 요청하는 것이다.**
> 

```graphql
{
  hero {
    name
  }
}
```

```json
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

☑️ 쿼리가 결과와 정확히 같은 모양을 하고 있음을 알 수 있는데, 이는 항상 예상한대로 돌아가고 서버는 클라이언트가 요구하는 필드를 정확히 알기 때문에 GraphQL에 필수적이다. 

☑️ 필드는 객체를 참조할 수도 있다. 이 경우, 해당 객체에 대한 필드의 sub-selection을 만들 수 있다.

☑️ GraphQL 쿼리는 관련 객체 및 해당 필드를 순회할 수 있으므로 클라이언트가 고전적인 REST 아키텍처에서 필요로 하는 여러 왕복을 하는 것과 반대로, **GraphQL은 하나의 요청으로 많은 관련 데이터를 가져올 수 있다.**

```graphql
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

## 🤓 인수 (Arguments)

> **GraphQL은 필드에 인수를 전달하는 기능이 있다.**
> 

```graphql
{
  human(id: "1000") {
    name
    height
  }
}
```

```json
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 1.72
    }
  }
}
```

☑️ REST와 같은 시스템에서는 단일 인수 집합 (요청의 쿼리 매개변수 및 URL)만 전달할 수 있다.

☑️ 반면, GraphQL에서는 모든 필드와 중첩 객체가 고유한 인수 집합을 얻을 수 있으므로 GraphQL을 여러 API 가져오기를 통해 완벽한 대체자가 될 수 있다.

## 🤨 별칭 (Alias)

> **별칭 사용을 통해 필드 결과의 이름을 원하는 이름으로 대체할 수 있다.**
> 

```graphql
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

```json
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```

## 🤩 조각 (Fragments)

> **Fragments를 사용하면 필드 집합을 구성한 후 필요한 쿼리에 포함할 수 있다.**
> 

<aside>
✅ **GraphQL에는 Fragments라는 재사용 가능한 단위가 포함되어 있다.**

</aside>

```graphql
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

```json
{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        },
        {
          "name": "C-3PO"
        },
        {
          "name": "R2-D2"
        }
      ]
    },
    "rightComparison": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

## Fragments 내에서 변수 사용하기

> **Fragments가 query 또는 mutation에 선언된 변수에 접근할 수 있다.**
> 

```graphql
query HeroComparison($first: Int = 3) {
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  friendsConnection(first: $first) {
    totalCount
    edges {
      node {
        name
      }
    }
  }
}
```

```json
{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "friendsConnection": {
        "totalCount": 4,
        "edges": [
          {
            "node": {
              "name": "Han Solo"
            }
          },
          {
            "node": {
              "name": "Leia Organa"
            }
          },
          {
            "node": {
              "name": "C-3PO"
            }
          }
        ]
      }
    },
    "rightComparison": {
      "name": "R2-D2",
      "friendsConnection": {
        "totalCount": 3,
        "edges": [
          {
            "node": {
              "name": "Luke Skywalker"
            }
          },
          {
            "node": {
              "name": "Han Solo"
            }
          },
          {
            "node": {
              "name": "Leia Organa"
            }
          }
        ]
      }
    }
  }
}
```

## 실제 프로젝트에 적용해보기

### graphql 관련 라이브러리 설치

`npm install @nestjs/graphql apollo-server-express graphql-tools type-graphql graphql`

### app.module.ts 구성

```tsx
@Module({
	imports: [
		...
		TypeOrmModule.forRoot({
      type: "mysql",
      host: process.env.DATABASE_HOST,
      port: 3306,
      username: process.env.DATABASE_USERNAME,
      password: process.env.DATABASE_PASSWORD,
      database: "test",
      entities: [__dirname + "/**/*.entity{.ts,.js}"],
      synchronize: true,
    }),
		GraphQLModule.forRoot({
      autoSchemaFile: "schema.gql",
    }),
		...
	]
})
```

☑️ entities 옵션을 통해 entity 파일들에 있는 모델들을 추가할 수 있다.

### Entity 구성

> typeorm에서 entity를 통해 미리 정의해둔 모델이 있다.
여기에 graphql은 typeDefs를 통해 데이터 타입과 요청의 타입을 정의하는 것을 추가해주자.
> 

<br/>

💡 **code first**
<aside>
→ graphQL schema를 만들기 위해 데코레이터와 클래스를 이용하는 방법이다. 

→ GraphQLModule의 autoSchemaFile을 true로 한 이유이다.
</aside>

```tsx
import { Column, Entity, PrimaryColumn } from "typeorm";
import { Field, Int, ObjectType } from "@nestjs/graphql";

@ObjectType()
@Entity("User")
export class UserEntity {
  @Field(() => Int)
  @PrimaryColumn()
  id: number;

  @Field(() => String)
  @Column({ length: 30 })
  name: string;

  @Field(() => String)
  @Column({ length: 60 })
  email: string;

  @Field(() => String)
  @Column({ length: 30 })
  password: string;

  @Field(() => String)
  @Column({ length: 60 })
  signupVerifyToken: string;
}
```

#### Options (Field, Column)

- nullable
- description

### Repository 활용하기

#### module에 등록하기

☑️ `user.module.ts`에 다음과 같이 forFeature 메서드를 이용하여 entity를 등록해준다.

☑️ 다른 module에서 해당 repository를 사용하려면 전체 module을 export해야 한다.

```tsx
import { Module } from "@nestjs/common";
import { EmailModule } from "src/email/email.module";
import { UsersService } from "./users.service";
import { UsersController } from "./users.controller";
import { TypeOrmModule } from "@nestjs/typeorm";
import { UserEntity } from "./entities/user.entity";

@Module({
  imports: [EmailModule, TypeOrmModule.forFeature([UserEntity])],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

### service provider에서 사용하기

☑️ module에 등록한 repository를 InjectRepository 데코레이터를 통해 사용할 수 있다.

```tsx
import { Injectable } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { Repository, DataSource } from "typeorm";
import { UserEntity } from "./entities/user.entity";

@Injectable()
export class UsersService {
  constructor(
		...
    @InjectRepository(UserEntity)
    private userRepository: Repository<UserEntity>,
    private dataSource: DataSource
  ) {}
  async createUser(name: string, email: string, password: string) {
    const userExist = await this.checkUserExists(email);
    if (userExist) {
      throw new UnprocessableEntityException(
        "해당 이메일로는 가입할 수 없습니다."
      ); // 422 error
    }

    const signupVerifyToken = uuid.v1();

    await this.saveUserUsingQueryRunner(
      name,
      email,
      password,
      signupVerifyToken
    );
    await this.sendMemberJoinEmail(email, signupVerifyToken);
  }

	...
```

### Resolver 생성하기

<aside>
💡 resolver는 요청들의 액션을 실제로 구현하는 부분이다.

</aside>

**☑️ graphQL에서 요청은 Query와 Mutation으로 나눠진다.**

- Query: CRUD에서 Read 담당
- Mutation: CRUD에서 Create, Update, Delete 담당

> typeDefs에서 요청이 어떤 params를 받아 어떤 것을 return할지 명시해준다면,
resolver에서는 실제 해당 함수를 구현하는 것이다.
> 

#### 회원가입 부분 graphql 구현해보기

```tsx
import { Field, InputType } from "@nestjs/graphql";

@InputType()
export class CreateUserInput {
  @Field()
  name: string;

  @Field()
  email: string;

  @Field()
  password: string;
}
```

```tsx
import {
  Resolver,
  Query,
  Args,
  Int,
  ResolveField,
  Parent,
  Mutation,
} from "@nestjs/graphql";
import { UserEntity } from "./entities/user.entity";
import { UsersService } from "./users.service";
import { CreateUserInput } from "./createUserInput";

@Resolver(() => UserEntity)
export class UsersResolve {
  constructor(private readonly usersService: UsersService) {}

  @Mutation(() => UserEntity)
  async createUser(@Args("createUserInput") createUserInput: CreateUserInput) {
    return await this.usersService.createUser(createUserInput);
  }
}
```

☑️ playground (`주소/graphql`)에서는 다음과 같이 작동시키면된다.

```
mutation {
  createUser(createUserInput: {
    name:"sylee1998",
		email: "sylee1998@test.com",
    password: "qwer1234"
  }) {
    id,
		name,
		email,
    password,
  }
}
```