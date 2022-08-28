# 당근 마켓 백엔드 클론코딩 Dev-log

# 개발 기간
2022.08.17 ~ 2022.

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