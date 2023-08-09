# Intersection Observer API

## InsersectionRect

![image](https://github.com/dltmddus1998/dltmddus1998.github.io/assets/73332608/9b5e21d5-0c7f-47c5-9f78-c9f711ba6270)

> 역사적으로 요소의 가시성이나 두 요소의 상대적인 가시성을 탐지하는 것은 어려운 일이었다. 일반적으로 알려진 해결방법은 신뢰성이 부족하고 브라우저나 사이트에 부하를 주기 때문에 좋지 못한 사용자 경험을 낳는다.
> 

<aside>
💡 웹이 성숙함에 따라 이러한 정보의 필요성은 커졌고, intersection 정보는 아래와 같은 여러가지 이유 때문에 필요하다

</aside>


- 페이지가 스크롤 되는 도중에 발생하는 이미지나 다른 컨텐츠의 지연 로딩
- 스크롤 시에, 더 많은 컨텐츠가 로드 및 렌더링되어 사용자가 페이지를 이동하지 않아도 되게 하는 infinite-scroll을 구현
- 광고 수익을 계산하기 위한 용도로 광고의 가시성 보고
- 사용자에게 결과가 표시되는 여부에 따라 작업이나 애니메이션을 수행할 지 여부 결정

### 복수 개의 라이브러리를 사용했을 때 발생하는 상황

infinite-scroll이 구현된 웹 페이지를 생각해보면, 공급 업체가 제공하는 라이브러리를 사용하여 페이지 전체에 주기적으로 배치되는 광고를 관리하고 애니메이션 그래픽이 있으머 알림 상자 등을 그리는 사용자 라이브러리를 사용한다. **요소들 각각은 자신만의 교차 감지 루틴이 존재하고 이 모든 것들은 메인 스레드 위에서 동작한다**. 

웹 페이지의 작성자는 사용중인 두 라이브러리의 내부 동작을 거의 알지 못하므로 이러한 일이 발생하는 것을 알지 못할 수도 있다. 사용자가 페이지를 스크롤할 때, 이러한 교차 탐지 루틴은 스크롤 처리 코드 중에 반복적으로 실행되므로 사용자는 브라우저, 웹사이트 및 컴퓨터에 좌절감을 느끼게 된다. 

### 이를 개선하기 위한 Intersection Observer API의 역할

Intersection Observer API는 그들이 감시하고자 하는 요소가 다른 요소(viewport)에 들어가거나 나갈 때 또는 요청한 부분만큼 두 요소의 교차 부분이 변경될 때마다 실행될 콜백 함수를 등록할 수 있게 한다.

→ **사이트는 요소의 교차를 지켜보기 위해 메인 스레드를 사용할 필요가 없어지고 브라우저는 원하는 대로 교차 영역 관리를 최적화할 수 있다.** 

ex) 정확히 몇 픽셀이 겹쳐졌고 어떠한 픽셀이 겹쳤는지  알려줄 수는 없지만, “N% 정도 교차하는 경우 상호작용이 이루어져야한다”와 같은 더 일반적인 사용 사례를 다룰 수 있다.

### Intersection Observer 생성

> intersection observer를 생성하기 위해서는 생성자 호출 시 콜백 함수를 제공해야 한다. (해당 콜백함수는 threshold가 한 방향 혹은 다른 방향으로 교차할 때 실행된다.)
> 

```jsx
let options = {
	root: document.querySelector('#scrollArea'),
	rootMargin: '0px',
	threshold: 1.0
}

let observer = new IntersectionObserver(callback, options)
```

`threshold: 1.0`은 대상 요소가 `root`에 지정된 요소 내에서 100% 보여질 때 콜백이 호출될 것을 의미한다.

### Intersection Observer 설정

`root`

대상 객체의 가시성을 확인할 때 사용되는 뷰포트 요소이다. 이는 대상 객체의 조상 요소여야 합니다. 기본값은 브라우저 뷰포트이며, `root` 값이 `null` 이거나 지정되지 않을 때 기본값으로 설정된다..

`rootMargin`

root 가 가진 여백이다. 이 속성의 값은 CSS의 `[margin](https://developer.mozilla.org/ko/docs/Web/CSS/margin)` 속성과 유사하다. e.g. "`10px 20px 30px 40px"` (top, right, bottom, left). 이 값은 퍼센티지가 될 수 있다. 이것은 root 요소의 각 측면의 bounding box를 수축시키거나 증가시키며, 교차성을 계산하기 전에 적용된다. 기본값은 0이다.

`threshold`

observer의 콜백이 실행될 대상 요소의 가시성 퍼센티지를 나타내는 단일 숫자 혹은 숫자 배열이다. 만일 50%만큼 요소가 보여졌을 때를 탐지하고 싶다면, 값을 `0.5`로 설정하면 된다. 혹은 25% 단위로 요소의 가시성이 변경될 때마다 콜백이 실행되게 하고 싶다면 `[0, 0.25, 0.5, 0.75, 1]` 과 같은 배열을 설정하면 된다. 기본값은 `0`이며(이는 요소가 1픽셀이라도 보이자 마자 콜백이 실행됨을 의미한다). `1.0`은 요소의 모든 픽셀이 화면에 노출되기 전에는 콜백을 실행시키지 않음을 의미한다.

### 사용 예시 코드

```jsx
const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
		// 타겟 요소가 루트 요소와 교차하는 지점이 없으면 콜백을 호출하되, 조기에 탈출 (!entry.isIntersecting)
		console.log(entry); // **💡 IntersectionObserversEntry 인스턴스의 배열**
    if (entry.isIntersecting) { // (entry.intersectionRatio > 0 이라고도 표현할 수 있다)
      entry.target.classList.add('show');
    } else {
      entry.target.classList.remove('show');
    }
  });
});

const hiddenElements = document.querySelectorAll('.hidden');
hiddenElements.forEach((el) => observer.observe(el));
```

**💡 IntersectionObserversEntry 인스턴스의 배열**

`entry`를 콘솔에 찍어보면 다음과 같이 나온다

![image](https://github.com/dltmddus1998/dltmddus1998.github.io/assets/73332608/ecc53474-9de8-41fc-8a9d-50ed892eeaac)


| 속성 |  |  |
| --- | --- | --- |
| boundingClientRect | 관찰 대상의 사각형 정보 |  |
| intersectionRect | 관찰 대상의 교차한 영역 정보 |  |
| intersectionRatio | 관찰 대상의 교차한 영역 백분율 | intersectionRect 영역에서 boudingClientRect 영역까지 비율 |
| isIntersecting | 관찰 대상의 교차 상태 (boolean) | 관찰 대상이 루트 요소와 교차 상태로 들어가거나 (true) 교차 상태에서 나가는지 (false) 여부를 나타내는 boolean값 |
| rootBounds | 지정한 루트 요소의 사각형 정보 | 옵션 rootMargin에 의해 값 변경
별도의 루트 요소 선언 x → null 반환 |
| target | 관찰 대상 요소 |  |
| time | 변경이 발생한 시간 정보 | https://developer.mozilla.org/en-US/docs/Web/API/DOMHighResTimeStamp 반환 |

![image](https://github.com/dltmddus1998/dltmddus1998.github.io/assets/73332608/a6bb318a-d641-4b10-9cca-6f0521a549b8)


![image](https://github.com/dltmddus1998/dltmddus1998.github.io/assets/73332608/075131a3-6d1b-4ac6-9589-2c4b93449489)


![image](https://github.com/dltmddus1998/dltmddus1998.github.io/assets/73332608/7b44065e-0ddc-4414-88b1-ce0e76d4fe15)

### Reference

https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API

[https://velog.io/@elrion018/실무에서-느낀-점을-곁들인-Intersection-Observer-API-정리](https://velog.io/@elrion018/%EC%8B%A4%EB%AC%B4%EC%97%90%EC%84%9C-%EB%8A%90%EB%82%80-%EC%A0%90%EC%9D%84-%EA%B3%81%EB%93%A4%EC%9D%B8-Intersection-Observer-API-%EC%A0%95%EB%A6%AC)

https://heropy.blog/2019/10/27/intersection-observer/
