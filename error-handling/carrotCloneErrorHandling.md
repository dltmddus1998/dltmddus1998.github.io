# Error Handling with 당근마켓 (1) - GraphQLError

## GraphQLError: Query root type must be provided

![image](https://user-images.githubusercontent.com/73332608/187089058-fb8c7a3b-7081-4c81-a4cc-deed6c45bf90.png)

> 구글링해본 결과 resolver에서 Query에 대한 코드가 빠져있다고 되어 있는데, 이상한 점은 지금 구현하고 있는 부분이 데이터베이스를 수정해야하는 부분이라 Mutation이 쓰는 것이 맞는데 이렇게 하면 계속 저 에러가 난다.
>

> 다행히 리팩토링 이후 해당 에러는 없어졌다. repository, service 파일을 구분한 것 외에는 Mutation 관련해서 변경된 부분은 없는데 이 부분은 아직 미스테리다...😭
>