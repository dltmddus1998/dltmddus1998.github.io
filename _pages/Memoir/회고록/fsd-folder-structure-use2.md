---
title: "FSD 폴더구조 사용 후기 (2)"
tags:
  - FSD
  - Feature-Sliced Design
  - 방법론
date: "2024-11-02"
thumbnail: "/assets/img/thumbnail/fsd.png"
bookmark: true
---

# 화면에서 FSD 폴더구조 바로 이해해보기

## 네이버를 예로 화면과 fsd 폴더구조를 연관지어 이야기해보자.

<aside>

오늘은 FSD 폴더구조를 실질적으로 어떻게 사용해야 효과적으로 사용할 수 있을지 화면단에서 설명해보고자 한다.

</aside>

우선 대충 네이버 화면을 보고 생각해보자

![image](https://github.com/user-attachments/assets/3c632e5a-2277-4fbd-896f-3d9d36f07669)

난 우선 파란색 박스 내에 있는 것들은 이 페이지 내에서 구성하는 요소들 중 가장 큰 단위로 분류했다.

그리고 pages는 widgets으로 이뤄져 있다고 생각해서 widgets으로 분류했다.

이제 이 widgets들이 무엇으로 이뤄져 있는지 확인해보자.

### 먼저 가장 위 검색창 부터 분석해보자.

**layer별로 분류하려면 먼저 구성 요소에 대해 생각해봐야한다.**

1. N 로고
2. 키보드 아이콘 모양 입력도구 버튼
3. 검색기록 내려보기 버튼
4. 검색 버튼
5. 검색창

크게 네가지로 분류되는데, 우선 1번 로고의 경우 클릭하면 현재 페이지인 home으로 가는 링크가 달려 있으므로 하나의 기능을 한다고 볼 수 있다.

2, 3, 4, 5모두 동일한 경우이다.

이전에 말했듯이 widgets에 들어가는 자잘한 기능들을 features

따라서 1, 2, 3, 4, 5는 모두 Layers 중 features에 해당한다.

근데 이제 여기서 4. 검색 버튼은 features 내부에서도 무언가 분류된다. 그 이유는 바로 검색 버튼을 누르면 일어나는 action 때문인데, “검색 버튼을 누르면 검색 창에 써져 있는 검색어를 통해 검색 요청이 백엔드로 간다”가 이 action이다.

그렇다면 백엔드에 요청이 가는 api는 어디에 포함되어야 할까? 이전에 말했던 entities 레이어에 다음과 같이 들어가게 된다. (내 프로젝트에 맞게 예시를 보여주겠다.)

```jsx
📁 entities
	📁 model
		types.ts - typescript를 사용했음 (api response관련 타입 정의)
		stores.ts - 상태 관리
	📁 api
		index.ts - 백엔드 api
```

## export 방식

프로젝트를 진행하면서 이 부분이 간단하지만 중요한 부분이라고 느꼈다.

그 이유를 예를 들어 설명해보겠다. 우선 widgets에 layouts이라는 폴더를 만들어서 대부분의 화면에서 사용되는 레이아웃들을 정의했다.

근데 만약에 각 레이아웃에 대해 단계별로 export 하지 않고 바로 해당 레이아웃을 쓰는 페이지에서 import해서 쓴다면 정확히 해당 레이아웃이 어떤 용도인지 파악하기 힘들 수 있다. 특히나 코드는 보통 혼자 쓰는게 아닌 여러명이 함께 협업하는 경우가 많기 때문에 더욱 중요하다.

```
📁 widgets
	📁 layouts
		📁 gnb
			index.ts 1️⃣
		📁 lnb
			index.ts
		📁 editForm
			index.ts
		index.ts 2️⃣
	index.ts 3️⃣
```

위의 경우 export를 총 세번에 거쳐서 하면 된다.

1️⃣ 에서 각 위젯의 화면파일을 import한 후 바로 export한다.

```
// index.ts
import GNBWidget from './ui/GNBWidget.xx'

export { GNBWidget }
```

2️⃣ 에서 바로 윗 단위 폴더들을 export 해준다.

```
export * from './gnb'
export * from './lnb'
export * from './editForm'
```

마지막으로 3️⃣ 에서 최종으로 export 해준다.

```
export * from './layouts'
```

> 예시만 보면 굳이 이렇게까지 복잡하게 해야하나 싶겠지만, widgets내부에는 프로젝트가 커질수록 점점 더 깊은 tree로 수십개의 폴더가 나오게 된다. 그렇게 되면 위처럼 하지 않았을 때 import한다면 그것만큼 지저분한 import가 없을것이다.

`import xxxwidget from ‘@/widgets/user/sign-in/ui/xxxwidget.xx’` 이런식으로…

### the end…

이상 내가 프로젝트를 진행하면서 이해한 fsd 폴더구조이다.

사실 공식문서를 따라 이해하려면 꽤 많은 시간이 걸리기 때문에 나 같은 경우에는 공식문서는 1회독 정도 가볍게 읽고 실제로 fsd 폴더구조를 사용한 프로젝트들을 최대한 참고하여 프로젝트를 진행했고, 실제로 중간 중간 수많은 시행착오를 겪으면서 나만의 fsd 폴더구조로 프로젝트를 완성할 수 있었던 것 같다.

공식문서대로 fm으로 이해하고 그대로 수행하는 것도 중요하지만 직접 구현하고 시행착오를 겪으며 여러 상황에 부딪혀보니 이 어려운 방법론에 좀 더 가까이 다가가 이해할 수 있었던 것 같다.

앞으로 새로운 기술 스택이나 방법론을 공부할 때 이론과 실습을 동시에 진행하는게 좀더 이해를 빠르고 정확하게 할 수 있다는 것을 깨달았다.
