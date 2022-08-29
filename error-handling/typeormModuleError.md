# Error Handling with 당근마켓 

## GraphQLError: Query root type must be provided

![image](https://user-images.githubusercontent.com/73332608/187089058-fb8c7a3b-7081-4c81-a4cc-deed6c45bf90.png)

> 구글링해본 결과 resolver에서 Query에 대한 코드가 빠져있다고 되어 있는데, 이상한 점은 지금 구현하고 있는 부분이 데이터베이스를 수정해야하는 부분이라 Mutation이 쓰는 것이 맞는데 이렇게 하면 계속 저 에러가 난다.
>

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
