# Error Handling with 당근마켓 (2) - NestExceptionError

## NestExceptionError: Nest can't resolve dependencies

![image](https://user-images.githubusercontent.com/73332608/187139045-9b9a954b-0a3d-468e-8799-ab7a6dd0e2b2.png)

> **위에 빨간색 에러를 보면 “index[1] 위치의 Repository가 Module에서 사용가능한지”에 대한 내용이다.**
> 

index[1]은 Service 클래스의 생성자에 주입한 두 번째 인자를 뜻하는데 현재의 나의 코드에서는 다음과 같다.

```tsx
...
constructor(
    @InjectRepository(PostEntity)
    private postRepository: Repository<PostEntity>,
    private priceOfferRepository: Repository<PriceOfferEntity>, ✔︎
    private connection: Connection, 
  ) {}
...
```

즉, Service 클래스에서 PriceOfferEntity에 대한 주입이 제대로 되지 않은 것 같다.

구글링 해보니 코드 구현이 완전히 잘못되었고 module 파일에서 Entity를 import해주는 코드도 잘못되었다.

### 잘못된 코드

```tsx
... // post.service.ts
constructor(
    @InjectRepository(PostEntity)
    private postRepository: Repository<PostEntity>,
    private priceOfferRepository: Repository<PriceOfferEntity>, ✔︎
    private connection: Connection, 
  ) {}
...

... // post.module.ts
imports: [TypeOrmModule.forFeature([PostEntity, PriceOfferEntity])],

...
```

위처럼 하면 주입이 제대로 안된다. 다음과 같이 독립적으로 진행해줘야한다.

### 옳은 코드

```tsx
... // post.service.ts
@InjectRepository(PostEntity)
private postRepository: Repository<PostEntity>,
@InjectRepository(PriceOfferEntity)
private priceOfferRepository: Repository<PriceOfferEntity>,
private connection: Connection,
...

... // post.module.ts
imports: [
	TypeOrmModule.forFeature([PostEntity]), 
	TypeOrmModule.forFeature([PriceOfferEntity])
	],
...
```

## GraphQLError: "message": "Expected Iterable, but did not find one for field

![image](https://user-images.githubusercontent.com/73332608/187159479-56980559-52bf-4e7c-9b8c-474a40e6de41.png)

🍀 계속해서 이런 에러가 나온다… 근데 진짜 nest.js + typeorm + graphql 이 셋다 자료가 너무 없어서 혼자 해결하기 너무 힘들다. 구글링이 거의 필요없고 그냥 공식문서에 의존해서 혼자 원시적인 것까지 다 생각해야해서 이게 진정한 개발인건가 라는 생각이 들었다…

> 💩 정말 아무것도 수정하지 않았는데 갑자기 된다.
> 

이유는 모르겠지만, 다음과 같이 잘 돌아간다. 서버는 —watch로 자동으로 새로고침되도록 해놓았기때문에 서버 재시작의 문제는 아닌듯 하고 잠시 graphql쪽에서 문제가 생겼던 것 같다. 

![image](https://user-images.githubusercontent.com/73332608/187159540-54c83645-ae3a-46c2-9c30-2f8fd43fbdd2.png)

