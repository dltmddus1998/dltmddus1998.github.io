# Error Handling with 당근마켓 

## GraphQLError: Query root type must be provided

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6c93d70b-d8c4-46b5-bcee-71eb9679c99e/Untitled.png)

> 구글링해본 결과 resolver에서 Query에 대한 코드가 빠져있다고 되어 있는데, 이상한 점은 지금 구현하고 있는 부분이 데이터베이스를 수정해야하는 부분이라 Mutation이 쓰는 것이 맞는데 이렇게 하면 계속 저 에러가 난다.
>