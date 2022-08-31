# 당근 마켓 백엔드 클론코딩 Dev-log

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

```tsx
// 게시글 끌올
@Mutation(() => PostEntity)
async pullupPost(@Args('pullUpPostInputDto') pullUpPostInputDto: PullUpPostInputDto) {
  return await this.postService.pullUpPost(pullUpPostInputDto);
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
async pullUpPost(pullUpPostInputDto: PullUpPostInputDto) {
  const {id} = pullUpPostInputDto;
  // console.log(postId);
  const post = await this.findPost(id);
  if (!post) {
    throw new NotFoundException('게시물을 찾을 수 없습니다.'); // 404 error
  }

  await this.changeUpdatedAt(id);
}
```

#### findPost (private 메서드)

```tsx
const post = await this.postRepository.findOne({
  where: {
    postId,
  },
});

return post;
```

#### changeUpdatedAt (private 메서드)

```tsx
const post = await this.postRepository.findOne({
  where: {
    postId,
  },
});

post.updatedAt = new Date();

await this.postRepository.save(post);

return post === undefined;
```

### post.module.ts 구현

```tsx
import {Module} from '@nestjs/common';
import {PostService} from './post.service';
import {PostResolver} from './post.resolver';
import {TypeOrmModule} from '@nestjs/typeorm';
import {PostEntity} from './post.entity';

@Module({
  imports: [TypeOrmModule.forFeature([PostEntity])],
  providers: [PostService, PostResolver],
})
export class PostModule {}
```

## 리팩토링 (+ 계층 분리)

> 협업중인 팀원과의 코드 비교 결과 service 파일에 모든 API 관련 코드를 적은 내 코드와 비교하여 팀원은 service파일과 repository파일을 구분지어서 코드를 짰기 때문에 더 계층 분리가 잘된 팀원의 코드 형식을 따르기로 결정했다.
++) 그외에도 자잘한 수정이 이루어졌다.
> 

### post.resolver.ts

```tsx
@Mutation(() => Post)
@UsePipes(ValidationPipe)
async pullupPost(@Args('pullUpPostInputDto') pullUpPostInputDto: PullUpPostInputDto) {
  return await this.postService.pullUpPost(pullUpPostInputDto);
}
```

### post.services.ts

```tsx
async pullUpPost(pullUpPostInputDto: PullUpPostInputDto) {
  const { postId } = pullUpPostInputDto;
  const found = await this.postRepository.findOne(postId);

  if (!found) {
    throw new NotFoundException(`postId가 ${postId}인 것을 찾을 수 없습니다.`);
  }

  this.postRepository.changePulled(postId);

  return this.getPostById(postId); // postId로 해당 post 찾는 메서드 따로 선언 (응답 확인용)
}
```

### post.repository.ts (service에서 사용하는 메서드 - Repository)

#### changedPulled

```tsx
export class PostRepository extends Repository<Post> {
	...
	async changePulled(postId: number): Promise<void> {
	  await getRepository(Post).createQueryBuilder('post').update(Post).set({ pulledAt: new Date() }).where('postId = :postId', { postId }).execute();
	}
	...
}
```

## GraphQL 실행 화면

![image](https://user-images.githubusercontent.com/73332608/187600638-ea0ec4c2-f525-4315-b23c-f536a408a3f0.png)
