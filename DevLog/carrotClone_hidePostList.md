# 당근 마켓 백엔드 클론코딩 Dev-log (5) - 숨긴 게시글 리스트 조회하기

## 👩🏻‍💻 구현 방법

**⚠️ 숨김 처리 해제의 경우 이전에 구현한 숨김 처리 구현의 정반대로 구현하면 되기 때문에 생략하겠다.**

### 🖐🏻 데코레이터 생성

> 해당 유저가 맞는지 인증해주는 getUser 데코레이터를 생성하여 이를 resolver에서 사용해주자.
> 

```tsx
// getUser.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { User } from '../user.entity';

export const GetUser = createParamDecorator((data: string, context: ExecutionContext): User => {
  const ctx = GqlExecutionContext.create(context);
  const user = ctx.getContext().req.user;
  return data ? user?.[data] : user;
});
```

### post.service.ts

✔︎ 당근 마켓을 보면 판매 내역에서 판매중, 거래완료, 숨김 총 이 세 가지로 나눠서 본인이 작성한 게시글 리스트를 조회할 수 있다.

```tsx
async getHiddenPostsList(user: User, hiddenPostsListDto: HiddenPostsListDto) {
  return await this.postRepository.getHiddenPostsList(user, hiddenPostsListDto);
}
```

### post.repository.ts

**post.service.ts에서 사용되는 getHiddenPostsList를 다음과 같이 구현했다.**

✔︎ Post에서 isHidden이 true인 것들만 불러오면 된다.

✔︎ 단, 해당 유저의 게시글들만 불러와야 하므로 andWhere를 이용하여 해당 사항을 구현했다.

✔︎ 신고처리 된 게시글을 삭제 처리되므로 reportHandling이 false인 것만 불러온다.

```tsx
async getHiddenPostsList(user: User, hiddenPostsListDto: HiddenPostsListDto) {
  const { perPage, page } = hiddenPostsListDto;

  return await getRepository(Post)
    .createQueryBuilder('Post')
    .where('isHidden = :isHidden', { isHidden: true })
    .andWhere('reportHandling = :reportHandling', { reportHandling: false })
    .andWhere('user = :user', { user })
    .orderBy('post.createdAt', 'DESC')
    .offset((page - 1) * perPage)
    .limit(perPage)
    .getMany();
}
```

### post.resolver.ts

**위에서 구현한 getUser 데코레이터를 여기서 사용하여 해당 사용자가 맞는지 인증받도록 한다.**

```tsx
// 숨김처리 리스트 조회
@Query(() => Post)
@UsePipes(ValidationPipe)
getHiddenPosts(@GetUser() user: User, @Args('hiddenPostsListDto') hiddenPostsListDto: HiddenPostsListDto) {
  return this.postService.getHiddenPostsList(user, hiddenPostsListDto);
}
```

## GraphQL 실행 화면