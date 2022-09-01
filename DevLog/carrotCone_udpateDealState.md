# 당근 마켓 백엔드 클론코딩 Dev-log (4) - 게시글 판매 상태 변경하기

## ⚒ 데이터베이스 구성

### dealState.entity.ts

```tsx
import { Field, ObjectType } from "@nestjs/graphql";
import { Post } from "src/posts/post.entity";
import {BaseEntity, Column, Entity, JoinColumn, OneToMany, PrimaryColumn} from 'typeorm';

@Entity()
@ObjectType()
export class DealState extends BaseEntity {
  @Field()
  @PrimaryColumn({type: 'int'})
  dealStateId!: number;

  @Field()
  @Column({type: 'varchar'})
  dealState!: string; 

  @OneToMany(type => Post, post => post.dealState, { eager: false })
  posts!: Post[];
}
```

#### dealState static data 삽입 (임시)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/54f4fa67-80bb-4bba-9ae7-76d187d17834/Untitled.png)

## 👩🏻‍💻 구현 방법

### post.service.ts

☑️ dealState static data로 거래 상태를 등록하는 기능이다.

☑️ 요청에 담긴 dealState 값(1, 2, 3)에 따라 해당 게시글의 판매 상태를 업데이트한다.

💡 **TODO - 판매자(게시글 작성자) 만 변경 가능 -> user부분 구현되면 추가 예정)**

```tsx
async updateDealState(postId: number, updateDealStateDto: UpdateDealStateDto): Promise<Post> {
  /**
   * @ 코드 작성자: 이승연
   * @ 기능: 거래 상태 변경
   * @ [dealState] static data -> 거래 상태 등록 ✔︎
   * @ default: 판매중 (1)
   * @ request에 담긴 state에 따라 예약중 || 거래완료로 변경
   * @ 💡 TODO -> 판매자(게시글 작성자만 가능 -> user부분 구현되면 추가 예정)
   */

  await this.postRepository.updateDealState(postId, updateDealStateDto);
  return await this.getPostById(postId);
}
```

### DTO 구현

```tsx
import { Field, InputType } from '@nestjs/graphql';
import { IsNotEmpty, IsNumber } from 'class-validator';
import { Post } from '../post.entity';
import { DealState } from 'src/dealStates/dealState.entity';

@InputType()
export class UpdateDealStateDto {
  @Field(() => Number)
  @IsNotEmpty()
  @IsNumber()
  dealState!: DealState;
}
```

### post.repository.ts

```tsx
async updateDealState(postId: number, updateDealStateDto: UpdateDealStateDto) {
  const { dealState } = updateDealStateDto;

  await getRepository(Post).createQueryBuilder('Post').update(Post).set({ dealState }).where('postId = :postId', { postId: postId }).execute();
}
```

### post.resolver.ts

```tsx
// 게시글 상태 변경
@Mutation(() => Post)
@UsePipes(ValidationPipe)
updateDealState(@Args('postId', ParseIntPipe) postId: number, @Args('updateDealStateDto') updateDealStateDto: UpdateDealStateDto): Promise<Post> {
  return this.postService.updateDealState(postId, updateDealStateDto);
}
```

## GraphQL 실행 화면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4da758fe-63ef-4ad1-a688-52e95b7c77a6/Untitled.png)