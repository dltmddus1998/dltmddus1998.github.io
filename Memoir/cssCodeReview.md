# CSS 관련 코드리뷰 후기

> 생각나는대로 계속 업데이트할 예정이다.
> 

## ✓ Feedback

1. 중앙 정렬은 position: absolute 웬만하면 사용 금지
    1. flexbox를 활용하는게 훨씬 실용적임 (디스플레이 크기가 달라도 모두 중앙 정렬됨)
        
        ```css
        display: flex;
        align-items: center;
        justify-content: center;
        ```
        
2. 최대한 적은 코드로 표현할 수 있도록 해야함 
    1. css 속성은 부모 → 자식 요소로 상속되기 때문에 공통된 요소는 부모 요소에 최대한 적는다.
    2. 만약 자식 요소 중 다른게 있으면 그건 자식요소에 쓰면 된다. (css는 덮어쓰기가 된다.)
3. width, height 요소의 경우 %, px은 거의 사용하지 않는다.
    1. 위에서 말했듯이 상속 특성을 활용하여 다음과 같이 코드를 짜면 훨씬 간편하고 깔끔하게 원하고자 하는 UI를 표현할 수 있다.
        
        `min-height: inherit;`
        
4. 중복되는 속성들은 root에 변수로 미리 선언해두어 @include를 통해 가져다 쓸 수 있다.
    
    `@include container;`
    

## 한줄 요약

flexbox를 애용해라