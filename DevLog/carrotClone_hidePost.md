# 당근 마켓 백엔드 클론코딩 Dev-log (2) - 게시글 숨김 처리하기

## 👩🏻‍💻 구현 방법

> 게시글 숨김 처리 부분에서는 추가적으로 DB를 설계할 부분이 없었다.
> 

☑️ 게시글 신고 처리 상태가 `reportHandling = true` 일 때 `isHidden = true` 로 할당하여 숨김 처리를 진행했다.

☑️ 게시글 전체 조회시 `isHidden = false` 인 것만 filtering 하면 될 것이다.

### post.service.ts

```tsx
async hidePost(postId: number) {
  const post = await Post.findOne({
    where: {
      postId,
    },
  });

  if (!post) {
    throw new NotFoundException(`postId가 ${postId}인 것을 찾을 수 없습니다.`);
  }

  await this.postRepository.updateHiddenState(postId);

  return await this.getPostById(postId);
}
```

### post.repository.ts

```tsx
async updateHiddenState(postId: number) {
  await getRepository(Post).createQueryBuilder('Post').update(Post).set({ isHidden: true }).where('postId = :postId', { postId }).execute();
}
```

### post.resolver.ts

```tsx
// 게시글 숨김 처리
@Mutation(() => Post)
@UsePipes(ValidationPipe)
hidePost(@Args('postId', ParseIntPipe) postId: number) {
  return this.postService.hidePost(postId);
}
```

## GraphQL 실행 화면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f286956a-6aa0-4c05-9db1-f6bce45e8950/Untitled.png)