# 당근 마켓 백엔드 클론코딩 Dev-log

# 개발 기간
2022.08.17 ~ 2022.

# 게시판 끌올(맨위로 끌어올리기) 구현

## ⚒ 데이터베이스 구성

> 아직 전체적인 데이터베이스가 구성되지 않았기 때문에 foreign key 등의 DB의 관계성에 대한 부분은 우선 제외하고 구현했다.
> 

```tsx
import {Column, Entity, PrimaryColumn} from 'typeorm';
import {Field, ObjectType} from '@nestjs/graphql';

@Entity()
@ObjectType()
export class PostEntity {
  @Field()
  @PrimaryColumn({nullable: false})
  postId: number;

  @Field()
  @Column({
    length: 30,
    nullable: false,
  })
  userName: string;

  @Field()
  @Column({
    length: 30,
    nullable: false,
  })
  title: string;

  @Field()
  @Column({
    length: 500,
    nullable: false,
  })
  content: string;

  @Field()
  @Column({
    nullable: false,
  })
  price: number;

  @Field()
  @Column({
    default: false,
    nullable: false,
  })
  isOfferedPrice: boolean;

  @Field()
  @Column({
    default: false,
    nullable: false,
  })
  isHidden: boolean;

  @Field()
  @Column({
    default: false,
    nullable: false,
  })
  reportHandling: boolean;

  @Field()
  @Column({
    default: 0,
    nullable: false,
  })
  likes: number;

  @Field()
  @Column({
    default: 0,
    nullable: false,
  })
  views: number;

  @Field()
  @Column({
    nullable: false,
    default: new Date(),
  })
  createdAt: Date;

  @Field()
  @Column({
    nullable: false,
    default: new Date(),
  })
  updatedAt: Date;
}
```

## 👩🏻‍💻 구현 방법

1️⃣ POST 테이블을 기본적으로 updatedAt을 기준으로 정렬한다는 가정 하에 구현한다.

2️⃣ POST 테이블에서 끌어올리고자 하는 게시글의 updatedAt을 현재 날짜로 수정한다.

### post.resolve.ts 구현

> service 파일에서 구현한 “게시글 끌어올리기”를 위한 메서드를 여기에서 사용한다. → `pullUpPost`
> 

Resolver와 M

```tsx
import {Resolver, Args, Mutation} from '@nestjs/graphql';
import {PostEntity} from './post.entity';
import {PostService} from './post.service';
import {PullUpPostInput} from './pullUpPostInput';

@Resolver(() => PostEntity)
export class PostResolve {
  constructor(private readonly postService: PostService) {}

  @Mutation(() => PostEntity)
  async changeUpdatedAt(@Args('pullUpPostInput') pullUpPostInput: PullUpPostInput) {
    return await this.postService.pullUpPost(pullUpPostInput);
  }
}
```

### pullUpPostInput.ts

> 계층간 데이터 교환 구현
> 

```tsx
import {Field, InputType} from '@nestjs/graphql';

@InputType()
export class PullUpPostInput {
  @Field()
  postId: number;
}
```

### post.service.ts 구현

> InjectRepository 데코레이터로 POST 테이블을 통해 Repository를 생성했다. (데이터베이스 조작시 사용)
> 

```tsx
import {PullUpPostInput} from './pullUpPostInput';
import {Injectable, NotFoundException} from '@nestjs/common';
import {InjectRepository} from '@nestjs/typeorm';
import {Repository, Connection, getManager} from 'typeorm';
import {PostEntity} from './post.entity';

@Injectable()
export class PostService {
  constructor(
    @InjectRepository(PostEntity)
    private postRepository: Repository<PostEntity>,
    private connection: Connection,
  ) {}

  async pullUpPost(pullUpPostId: PullUpPostInput) {
    const {postId} = pullUpPostId;
    const post = await this.findPost(postId);
    if (!post) {
      throw new NotFoundException('게시물을 찾을 수 없습니다.'); // 404 error
    }

    await this.changeUpdatedAt(postId);
  }
	...
}
```

#### findPost (private 메서드)

```tsx
// pullUpPost에서 사용할 private 메서드 생성
private async findPost(postId: number): Promise<object> {
  try {
    const post = await this.postRepository.findOne({
      where: {
        postId,
      },
    });

    return post;
  } catch (err) {
    console.error(err);
    //   return {};
  }
}
```

#### changeUpdatedAt (private 메서드)

```tsx
private async changeUpdatedAt(postId: number): Promise<boolean> {
  try {
    await this.connection.transaction(async manager => { // transaction 처리
      const post = await this.postRepository.findOne({
        where: {
          postId,
        },
      });

      post.updatedAt = new Date();

      await manager.save(post);

      return post === undefined;
    });
  } catch (err) {
    console.error(err);
    return err;
  }
}
```

### post.app.module 구현

```tsx
import {Module} from '@nestjs/common';
import {PostService} from './post.service';
import {PostResolve} from './post.resolver';
import {TypeOrmModule} from '@nestjs/typeorm';
import {PostEntity} from './post.entity';

@Module({
  imports: [TypeOrmModule.forFeature([PostEntity])],
  controllers: [PostResolve],
  providers: [PostService],
})
export class PostModule {}
```

# 가격 제안하기 구현

# ⚒ 데이터베이스 구성

```tsx
import {Column, Entity, PrimaryColumn} from 'typeorm';
import {Field, Int, ObjectType} from '@nestjs/graphql';

@Entity()
@ObjectType()
export class PriceOfferEntity {
  @Field(type => Int)
  @PrimaryColumn({nullable: false})
  priceOfferId: number;

  @Field()
  @Column({
    default: 0,
    nullable: false,
  })
  offerPrice: number;

  @Field()
  @Column({
    default: false,
    nullable: false,
  })
  accept: boolean;

  @Field()
  @Column({
    default: new Date(),
    nullable: false,
  })
  createdAt: Date;
}
```

## 모델간 관계 설정 (postEntity ↔ priceOfferEntity)

☑️ 게시글 1개당 가격 제안은 여러개 존재할 수 있으므로 이는 **1:N 관계**로 표현할 수 있다.

→ typeorm의 **OneToMany**와 **ManyToOne**을 이용하여 구현해보자.

### PostEntity (1)

```tsx
...
@OneToMany(type => PriceOfferEntity, priceOfferEntity => priceOfferEntity.postEntities)
  priceOfferEntities!: PriceOfferEntity[];
...
```

### PriceOfferEntity (N)

```tsx
...
@ManyToOne(type => PostEntity, postEntity => postEntity.priceOfferEntities)
  postEntities!: PostEntity;
...
```

### ✔︎ 데이터베이스 구조 확인

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/79a4a5c5-f0e2-4894-bf87-2057cc5bf60e/Untitled.png)

# 👩🏻‍💻 기능 구현

## post.service.ts 구현

↙️ 이전에 구현했던 파일에 이어서 구현했다.   

[게시글 끌어올리기 기능 구현](https://www.notion.so/278f59e83dc04b908153a496fbf02407)

### offerPrice (가격 제안 메서드)

1️⃣ 가격 제안 요청 to 판매자 (제안하고자 하는 가격, 판매자 PUT) ()

2️⃣ 판매자에게 가격 제안 알림 기능 ()

🅰️ 수락시 - isOfferedPrice = true & price = offeredPrice로 재할당 ()

🅱️ 거절시 - nothing ()

☑️ 데이터 교환을 위해 만들어준 offerPriceDto 타입으로 인수를 받아온다.

```tsx
import {Field, InputType} from '@nestjs/graphql';

@InputType()
export class OfferPriceDto {
  @Field()
  readonly priceOfferId: number;

  @Field()
  readonly postId: number;

  @Field()
  offerPrice: number;

  @Field()
  accept: boolean;
}
```

☑️ typeorm의 connection을 통해 데이터베이스 트랜잭션 처리를 했다. 

☑️ 예외처리는 `@nestjs/common`의 `NotFoundException`을 통해 404 에러 처리를 진행했다.

☑️ private으로 offerPrice에서 사용할 메서드들을 선언해주었다. 

```tsx
async offerPrice(offerPriceDto: OfferPriceDto): Promise<boolean> {
  const {priceOfferId, postId, offerPrice, accept} = offerPriceDto;

  const queryRunner = this.connection.createQueryRunner();

  await queryRunner.connect();
  await queryRunner.startTransaction(); // 트랜잭션 처리

  const priceOfferedPost = await this.requestPriceToSeller(priceOfferId, offerPrice);

  if (!priceOfferedPost) {
    throw new NotFoundException('게시물을 찾을 수 없습니다.'); // 404 error
  }

  await this.responsePriceToSeller(); // TODO - 유저 테이블 구체화된 후 수정 (알림기능)

  const answer = await this.determineOfferedPrice(accept, priceOfferId, postId);

  queryRunner.manager.save(priceOfferedPost);

  return answer; // true || false
}
```

### requestPriceToSeller (판매자에게 가격 제안 요청)

```tsx
private async requestPriceToSeller(priceOfferId: number, offerPrice: number): Promise<object> {
  const priceOffered = await this.priceOfferRepository.findOne({
    where: {
      priceOfferId,
    },
  });

  priceOffered.offerPrice = offerPrice;
  this.priceOfferRepository.save(priceOffered);
  return priceOffered;
}
```

### 판매자에게 가격 제안 알림 보내기 (responsePriceToSeller - TODO)

```tsx
private async responsePriceToSeller() {}
```

### 판매자의 가격 제안 수락 여부 (determineOfferedPrice)

```tsx
private async determineOfferedPrice(accept: boolean, priceOfferId: number, postId: number): Promise<boolean> {
  const post = await this.postRepository.findOne({
    where: {
      postId,
    },
  });
  const priceOffered = await this.priceOfferRepository.findOne({
    where: {
      priceOfferId,
    },
  });
  if (accept) {
    post.isOfferedPrice = true;
    post.price = priceOffered.offerPrice;

    this.postRepository.save(post);
    this.priceOfferRepository.save(priceOffered);

    return true;
  } else {
    return false;
  }
}
```

☑️ **typeorm의 connection을 이용하여 트랜잭션을 처리해주었다.** 

```tsx
async offerPrice(offerPriceDto: OfferPriceDto): Promise<boolean> {
	...
  const queryRunner = this.connection.createQueryRunner();

  await queryRunner.connect();
  await queryRunner.startTransaction(); // 트랜잭션 처리
  ...
	queryRunner.manager.save(priceOfferedPost); // 저장

	...
}
```

## post.resolver.ts 구현

```tsx
// 가격 제안 to 판매자
@Mutation(() => PriceOfferEntity)
async offerPriceToSeller(@Args('offerPriceDto') offerPriceDto: OfferPriceDto): Promise<PriceOfferEntity> {
  return await this.postService.offerPrice(offerPriceDto);
}
```

## GraphQL로 request & response 확인하기

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1d249d76-be05-4517-9e02-be142ed5dee0/Untitled.png)