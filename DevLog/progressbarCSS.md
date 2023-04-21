# Progress Bar를 Animation으로 구현해보기

> css를 입사해서 주먹구구식으로 급하게 독학하다보니 애니메이션 쪽에서는 완전 잼병이다…🤭 그래서 이번 기회에 확실히 짚고 넘어가려고 한다.
> 

## CSS - 애니메이션

<aside>
👉🏻 먼저 애니메이션에 사용되는 css 기능들이 뭐가 있는지 살펴보았다.

</aside>

- **`@keyframes [Animation Name] {}`**
    
    **✓ 개발자가 애니메이션 중간중간의 특정 지점들을 거칠 수 있는 키프레임들을 설정함으로써 CSS 애니메이션 과정의 중간 절차를 제어할 수 있게 한다.**
    
- **`animation`**
    
    **✓ 다수의 스타일을 전환하는 애니메이션을 적용한다.**
    
- **`transform`**
    
    **✓ 요소에 회전, 크기 조절, 기울이기, 이동 효과를 부여할 수 있다.**
    
- **`transition`**
    
    **✓ 엘리먼트 두 가지 상태 사이에 변화를 줄 수 있다.**
    

더 많은 속성들이 있겠지만, 우선 업무가 급하기 때문에 딱 필요한 것들만 간단히 공부하고 바로 구현에 들어갔다.

## 구현하려는 것

> 구현하려는 Progress Bar는 아래 그림과 같은 타임라인 형식이다.
> 

![image](https://user-images.githubusercontent.com/73332608/233573151-43405116-8779-4604-8a91-fad84310b638.png)

일단 감이 안잡혀서 코드펜이나 코드샌드박스에서 서치해봤는데 비슷한 것 조차도 나오지 않았다… 나오더라도 유저가 액션을 취해야 다음으로 넘어가는 타임라인 애니메이션만 존재했다.

## 코드

정말 모르겠다 싶어서 일단 단순 progress bar부터 구현했다. (vue3 script setup 버전으로 구현했다.)

```html
<!-- HTML 코드 -->
<div class="progress">
  <div class="progress-bar__container">
    <div class="progress-bar"></div>
  </div>
</div>
```

```scss
// css 코드 (scss)
.progress {
  display: absolute;
  top: 100px;
  height: 100%;
  margin-left: 30px;
  &-bar {
    position: absolute;
    width: 100%;
    height: 100%;
    content: '';
    background-color: #182650;
    &__container {
      top: 30px;
      width: 12px;
      height: 270px;
      border-radius: 2rem;
      position: relative;
      overflow: hidden;
      transition: all 0.5s;
      will-change: transform;
      box-shadow: 0 0 5px #e1ffb1;
      background-color: #ffffff;
    }
  }
}
```

```tsx
// script 코드
onMounted(() => {
	const progressBar = document.querySelector('.progress-bar');
  const progressBarStates = [0, 25, 50, 75, 100];

  // const progressBarStates = [100, 95, 80, 68, 34, 27, 7, 0];
  let time = 0;
  let endState = 100;
	
  progressBarStates.forEach((state) => {
    let randomTime = Math.floor(Math.random() * 3000);
	  setTimeout(() => {    
	      if (state == endState) {
	        gsap.to(progressBar, {
	          y: `${state}%`,
	          duration: 2,
	          backgroundColor: '#000000',
	        });
	      } else {
	        gsap.to(progressBar, {
	          y: `${state}%`,
	          duration: 2,
	        });
	      }
	    }, randomTime + time);
	    time += randomTime;
	  });
	});
})
```

> 순수 css로 구현하는 것보다 gsap과 같은 좋은 css 라이브러리가 있기 때문에 gsap을 사용했다. 정말 항상 느끼지만 이런 라이브러리 만든 사람들은 상받아야 한다… (이미 받았나?)
> 

**✓** **setTimeout을 통해 초당 진행률을 쉽게 구현할 수 있었다.**

어쨌든 위와 같이 구현하면 아래와 같은 결과물이 나온다.

![image](https://user-images.githubusercontent.com/73332608/233573182-8bed4ab7-ff3f-4597-a8ea-a39189793082.png)

→ 캡쳐본이지만 애니메이션이 잘 구현된다. 하얀색이 점차 내려오는 형태의 bar progress가 구현됐다.

👉🏻 곧 이어서 타임라인을 위한 circle progress animation도 구현해볼 예정이다. 사실 지금 하고 있는데 생각보단 진전이 좋다. 아직 완성률은 30%이다…😭
