# 당근 마켓 백엔드 클론코딩 Dev-log (3) - 게시글 신고하기

## ⚒ 데이터베이스 구성

### 신고 이유 static data 저장

<aside>
💡 이유는 모르겠으나 typeorm-seeder가 제대로 동작하지 않아 기능 구현이 우선이라 생각되는 지금은 우선 API로 따로 데이터를 insert했다. 나중에 해결할 것.

</aside>

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c8d84a89-e5ed-4aa9-9896-021d787a9d2e/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8fe6c740-8fb8-47c0-b03f-3d4e3ec049f7/Untitled.png)

### postsComplaint.entity.ts

✔︎ **Foreign Key 구현을 위한 다른 테이블들과의 참조 구현**

```tsx
import { Field, Int, ObjectType } from '@nestjs/graphql';
import { BaseEntity, Column, CreateDateColumn, Entity, JoinColumn, ManyToOne, OneToMany, OneToOne, PrimaryGeneratedColumn, UpdateDateColumn } from 'typeorm';
import { Post } from './post.entity';
import { ProcessState } from '../../processStates/processState.entity';
import { ComplaintReason } from 'src/complaintReasons/complaintReason.entity';

@Entity()
@ObjectType()
export class PostsComplaint extends BaseEntity {
  @Field()
  @PrimaryGeneratedColumn({ type: 'int' })
  complaintId!: number;

  @Field(type => Post)
  @JoinColumn({ name: 'postId' })
  @ManyToOne(type => Post, post => post.postsComplaint, { eager: true, onUpdate: 'CASCADE', onDelete: 'CASCADE' })
  post!: Post;

  @Field()
  @JoinColumn({ name: 'complaintReasonId' })
  @ManyToOne(type => ComplaintReason, complaintReason => complaintReason.postsComplaint, { eager: true, onUpdate: 'CASCADE', onDelete: 'CASCADE' })
  complaintReason!: ComplaintReason;

  @Field()
  @JoinColumn({ name: 'processStateId' })
  @ManyToOne(type => ProcessState, processState => processState.postsComplaint, { eager: true, onUpdate: 'CASCADE', onDelete: 'CASCADE' })
  processState!: ProcessState;

  @Field()
  @Column({ type: 'text', nullable: true })
  memo!: string;

  @Field()
  @CreateDateColumn({ type: 'datetime' })
  createdAt!: Date;
}
```

## 👩🏻‍💻 구현 방법

1️⃣ PostsComplaints entity에 등록 → `createPostsComplaints.dto.ts` 생성

사용자 참조 부분은 user 관련 구현이 완료된 후 추가적으로 구현할 계획

```tsx
import { Field, InputType } from '@nestjs/graphql';
import { IsBoolean, IsNotEmpty, IsNumber, IsString, Length } from 'class-validator';
import { Post } from '../post.entity';
import { ComplaintReason } from 'src/complaintReasons/complaintReason.entity';
import { ProcessState } from 'processStates/processState.entity';

@InputType()
export class CreatePostsComplaintsDto {
  @Field(() => Number)
  @IsNotEmpty()
  @IsNumber()
  post!: Post;

  @Field(() => Number)
  @IsNotEmpty()
  @IsNumber()
  complaintReason!: ComplaintReason;

  @Field(() => Number)
  @IsNotEmpty()
  @IsNumber()
  processState!: ProcessState;

  //   @Field()
  //   @IsNotEmpty()
  //   @IsString()
  //   complaintUserId!: number;
}
```

2️⃣ 작성자 외 모든 사용자들이 해당 게시글 신고 가능 (신고요청)

### post.service.ts

```tsx
async reportPost(createPostsComplaintDto: CreatePostsComplaintsDto): Promise<PostsComplaint> {
  const insertId = await this.postRepository.createPostsComplaint(createPostsComplaintDto);

  return this.getPostsComplaintById(insertId);
}
```

### post.repository.ts

☑️ PostsComplaint entity에 데이터를 입력한다. 

☑️ 이 때 dto를 통해 전달할 데이터들을 표현했는데 processState은 기본값이 1이라고 가정한다.

```tsx
async createPostsComplaint(createPostsComplaintsDto: CreatePostsComplaintsDto): Promise<number> {
  const { post, complaintReason, processState } = createPostsComplaintsDto;

  const query = await getRepository(PostsComplaint)
    .createQueryBuilder('PostsComplaint')
    .insert()
    .into(PostsComplaint)
    .values({
      post,
      complaintReason,
      processState,
    })
    .execute();

  return query.raw.insertId;
}
```

## GraphQL 실행 화면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b8ea765c-2b5c-4c0b-a401-199f76a24f1f/Untitled.png)