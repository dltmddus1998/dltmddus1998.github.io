# 당근 마켓 백엔드 클론코딩 Dev-log (7) - 팔로우 및 모아보기 구현하기

## ⚒ 데이터베이스 구성

### followings.entity.ts

> User 테이블을 참조하여 팔로워 - 팔로잉하는 유저 관련 필드를 구성했다.
> 

```tsx
import { Field, ObjectType } from '@nestjs/graphql';
import { BaseEntity, CreateDateColumn, Entity, JoinColumn, ManyToOne, PrimaryGeneratedColumn } from 'typeorm';
import { User } from '../../users/entities/user.entity';

@Entity()
@ObjectType()
export class Followings extends BaseEntity {
  @Field()
  @PrimaryGeneratedColumn({ type: 'int' })
  followingId: number;

  @Field(() => User)
  @JoinColumn({ name: 'userPhoneNumber' })
  @ManyToOne(type => User, user => user.followings, { eager: true, onUpdate: 'CASCADE', onDelete: 'CASCADE' })
  followingUser!: User;

  @Field(() => User)
  @JoinColumn({ name: 'subjectUserPhoneNumber' })
  @ManyToOne(type => User, user => user.followers, { eager: true, onUpdate: 'CASCADE', onDelete: 'CASCADE' })
  followerUser!: User;

  @Field()
  @CreateDateColumn({ type: 'datetime' })
  createdAt!: Date;
}
```

#### 🚩 DB

1️⃣ **팔로워 테이블**

채팅방 ID[PK], 게시글 ID[FK], 메시지 발송자 (구매희망자)[FK], 생성일자, 신고처리, 판매자 삭제, 발송자 삭제

### 기능별 정리

#### 다른 유저 팔로우하기

✔︎ 예외처리

1. 본인을 팔로우하는 경우
2. 존재하지 않는 유저일 경우
3. follwingId에 해당하는 팔로잉이 없을 경우 (팔로우 처리가 제대로 되지 않음)

```tsx
async followUsers(user: User, followDto: FollowDto): Promise<Followings> {
  const { followerUser } = followDto;
  const followerUserFind = await User.findOne({ where: { phoneNumber: followerUser } });

  if (followerUser.toString() === user.phoneNumber) {
    throw new BadRequestException(`본인을 팔로우할 수 없습니다.`);
  } else if (!followerUserFind) {
    throw new BadRequestException(`존재하지 않는 유저입니다.`);
  } else {
    const insertId = await this.mypageRepository.followUsers(user, followerUser);
    const found = await Followings.findOne(insertId);
    if (!found) {
      throw new NotFoundException(`${insertId}에 해당하는 팔로우 기록이 없습니다.`);
    } else {
      return await this.getFollowingsById(insertId);
    }
  }
}
```

```tsx
// followUsers 메서드
async followUsers(user: User, followerUser: User): Promise<number> {
  const query = await getRepository(Followings)
			.createQueryBuilder('Followings')
			.insert()
			.into(Followings)
			.values({ followingUser: user, followerUser })
			.execute();
  return query.raw.insertId;
}

// getFolowingsById 메서드
async getFollowingsById(followingId: number): Promise<Followings> {
  const found = await Followings.findOne(followingId);
  if (!found) {
    throw new NotFoundException(`followingId가 ${followingId}인 것을 찾을 수 없습니다.`);
  }
  return found;
}
```

#### 팔로우 취소하기

✔︎ 예외처리

1. 본인이 아닌 경우 
2. follwingId에 해당하는 팔로잉이 없을 경우

```tsx
async deleteFollowUsers(user: User, followingId: number): Promise<String> {
  const following = await this.getFollowingsById(followingId);
  if (JSON.stringify(following.followingUser) !== JSON.stringify(user)) {
    throw new BadRequestException(`본인의 경우에만 팔로우를 취소할 수 있습니다.`);
  }
  const result = await this.mypageRepository.deleteFollowUsers(followingId);
  if (result.affected === 0) {
    throw new NotFoundException(`followingId가 ${followingId}인 것을 찾을 수 없습니다.`);
  }
  return '삭제되었습니다.';
}
```

```tsx
// deleteFollowUsers 메서드
async deleteFollowUsers(followingId: number) {
  return await getRepository(Followings)
		.createQueryBuilder('Followings')
		.delete()
		.from(Followings)
		.where('followingId = :followingId', { followingId })
		.execute();
}
```

#### 팔로잉 모아보기

```tsx
async seeFollowUsers(user: User, page: number, perPage: number): Promise<Followings[]> {
  /**
   * 팔로잉 모아보기
   *
   * @author 이승연(dltmddus1998)
   * @param {page, perPage} 조회할 페이지, 한 페이지당 게시글 개수
   * @returns {Followings[]} 팔로우 리스트 전체 반환
   */
  return await this.mypageRepository.seeFollowUsers(user, page, perPage);
}
```

```tsx
// seeFollowUsers 메서드
async seeFollowUsers(user: User, page: number, perPage: number) {
  return await getRepository(Followings)
    .createQueryBuilder('followings')
    .innerJoinAndSelect('followings.followingUser', 'user')
    .innerJoinAndSelect('followings.followerUser', 'subjectUser')
    .where('user.phoneNumber = :phoneNumber', { phoneNumber: user.phoneNumber })
    .orderBy('followings.createdAt', 'DESC')
    .offset((page - 1) * perPage)
    .limit(perPage)
    .getMany();
}
```

> 팔로잉 모아보기 기능의 경우, User 테이블을 참조한 Following 테이블의 특성을 이용한 조인을 통해 데이터를 조회했다.
> 

> WHERE절에서 해당 사용자의 PK인 phoneNumber을 통해 팔로잉하고 있는 유저들을 모아볼 수 있도록 구현했다.
> 

#### mypage.resolver.ts

```tsx
// 팔로우
@Mutation(() => Followings)
followUsers(@GetUser() user: User, @Args('followDto') followDto: FollowDto): Promise<Followings> {
  return this.mypageService.followUsers(user, followDto);
}

// 팔로우 취소
@Mutation(() => String)
deleteFollowUsers(@GetUser() user: User, @Args('followingId') followingId: number): Promise<String> {
  return this.mypageService.deleteFollowUsers(user, followingId);
}

// 팔로잉 모아보기
@Query(() => [Followings])
seeFollowUsers(@GetUser() user: User, @Args('page') page: number, @Args('perPage') perPage: number): Promise<Followings[]> {
  return this.mypageService.seeFollowUsers(user, page, perPage);
}
```

## graphQL

### 팔로우

#### 쿼리

```json
mutation {
  followUsers(followDto: {
    followerUser: "나나"
  }) {
    followingId,
    followingUser{
      userName
    }
    followerUser {
      userName
    }
    createdAt,
  }
}
```

#### 결과

```json
{
  "data": {
    "followUsers": {
      "followingId": 37,
      "followingUser": {
        "userName": "이승연"
      },
      "followerUser": {
        "userName": "나나"
      },
      "createdAt": "2022-09-15T06:47:23.382Z"
    }
  }
}
```

### 팔로잉 모아보기

```
import { Field, ObjectType } from '@nestjs/graphql';
import { BaseEntity, CreateDateColumn, Entity, JoinColumn, ManyToOne, PrimaryGeneratedColumn } from 'typeorm';
import { User } from '../../users/entities/user.entity';

@Entity()
@ObjectType()
export class Followings extends BaseEntity {
  @Field()
  @PrimaryGeneratedColumn({ type: 'int' })
  followingId: number;

  @Field(() => User)
  @JoinColumn({ name: 'userPhoneNumber' })
  @ManyToOne(type => User, user => user.followings, { eager: true, onUpdate: 'CASCADE', onDelete: 'CASCADE' })
  followingUser!: User;

  @Field(() => User)
  @JoinColumn({ name: 'subjectUserPhoneNumber' })
  @ManyToOne(type => User, user => user.followers, { eager: true, onUpdate: 'CASCADE', onDelete: 'CASCADE' })
  followerUser!: User;

  @Field()
  @CreateDateColumn({ type: 'datetime' })
  createdAt!: Date;
}
```

#### 쿼리

```json
query {
  seeFollowUsers(page: 1, perPage: 3) {
    followingId,
    followingUser {
      userName
    }
    followerUser{
			userName
    }
    createdAt
  }
}
```

#### 결과

```json
{
  "data": {
    "seeFollowUsers": [
      {
        "followingId": 39,
        "followingUser": {
          "userName": "이승연"
        },
        "followerUser": {
          "userName": "다다"
        },
        "createdAt": "2022-09-15T07:11:34.478Z"
      },
      {
        "followingId": 38,
        "followingUser": {
          "userName": "이승연"
        },
        "followerUser": {
          "userName": "나나"
        },
        "createdAt": "2022-09-15T07:10:14.530Z"
      },
    ]
  }
}
```