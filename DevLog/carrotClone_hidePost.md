# 당근 마켓 백엔드 클론코딩 Dev-log (2) - 게시글 숨김 처리하기

## 👩🏻‍💻 구현 방법

> 게시글 숨김 처리 부분에서는 추가적으로 DB를 설계할 부분이 없었다.
> 

☑️ 본인이 작성한 게시글을 숨김처리 및 해제를 할 수 있다.

☑️ 해당 사용자가 본인인지 인증해야 하므로 관련 데코레이터를 따로 생성해준다.

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

**여기에 사용되는 메서드들은 repository 파일에서 따로 구현해줬다.**

✔︎ 해당 게시글이 본인이 작성한 게시글이 아닐 경우 예외 처리를 해준다.

✔︎ 해당 게시글이 없을 경우 또한 예외 처리를 해준다.

```tsx
// 숨김 처리
async hidePost(user: User, postId: number) {
  const post = await this.getPostById(postId);
  if (JSON.stringify(post.user) !== JSON.stringify(user)) {
    throw new BadRequestException(`본인이 작성한 게시글만 숨김처리할 수 있습니다.`);
  }

  if (!post) {
    throw new NotFoundException(`postId가 ${postId}인 것을 찾을 수 없습니다.`);
  }

  await this.postRepository.updateHiddenStateTrue(postId);
  return await this.getPostById(postId);
}

// 숨김 처리 해제
async clearHiddenPostState(user: User, postId: number) {
  const post = await this.getPostById(postId);
  if (JSON.stringify(post.user) !== JSON.stringify(user)) {
    throw new BadRequestException(`본인이 작성한 게시글만 숨김처리를 해제할 수 있습니다.`);
  }

  if (!post) {
    throw new NotFoundException(`postId가 ${postId}인 것을 찾을 수 없습니다.`);
  }

  await this.postRepository.updateHiddenStateFalse(postId);
  return await this.getPostById(postId);
}
```

### post.repository.ts

**isHidden의 상태 변경을 통해 구현했다.**

```tsx
// 숨김 처리
async updateHiddenStateTrue(postId: number) {
  await getRepository(Post)
		.createQueryBuilder('Post')
		.update(Post)
		.set({ isHidden: true })
		.where('postId = :postId', { postId })
		.execute();
}

// 숨김 처리 해제
async updateHiddenStateFalse(postId: number) {
  await getRepository(Post)
		.createQueryBuilder('Post')
		.update(Post)
		.set({ isHidden: false })
		.where('postId = :postId', { postId })
		.execute();
}
```

### post.resolver.ts

```tsx
// 게시글 숨김 처리
@Mutation(() => Post)
@UsePipes(ValidationPipe)
hidePost(@GetUser() user: User, @Args('postId', ParseIntPipe) postId: number) {
  return this.postService.hidePost(user, postId);
}

// 게시글 숨김 처리 해제
@Mutation(() => Post)
@UsePipes(ValidationPipe)
clearHiddenPostState(@GetUser() user: User, @Args('postId', ParseIntPipe) postId: number) {
  return this.postService.clearHiddenPostState(user, postId);
}
```

## GraphQL 실행 화면

![image](https://user-images.githubusercontent.com/73332608/187749356-5d4d6d48-1049-4195-965a-55c32f48e7eb.png)
