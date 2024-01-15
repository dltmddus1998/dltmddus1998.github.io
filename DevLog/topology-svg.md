# SVG 활용하여 토폴로지 제작하기

## 토폴로지 구현 목적 (+ d2로 계속 구현하지 않는 이유)

> d2로는 디자인 쪽으로 한계가 너무 많음 → 레이아웃의 일정함 정도도 너무 떨어지고 동적인 데이터를 받았을 때 각 노드들의 위치 변경에 대한 변수를 예상할 수 없어서 다른 방법에 대해 리서치 중이다.
> 

🖱️ **d2관련 기술 블로그**

- [D2로 토폴로지 구현하기](https://dltmddus1998.github.io/DevLog/d2-topo1.html?highlight=d2)
- [NodeJS로 d2 실행하여 API를 통해 브라우저로 토폴로지 구현하기](https://dltmddus1998.github.io/DevLog/d2-topo4.html?highlight=d2)

> 그 중 한 가지 방법이 XD를 통해 디자인한 토폴로지를 각 노드들이 인식 가능하도록 svg로 추출한 후 해당 svg파일로 동적 데이터를 직접 조작하는 방법이다.
> 

💡 **일단 가장 간단한 예로 노드간의 링크를 구현해야하는 토폴로지로 구현을 해보았는데, `leader-line`이라는 npm의 라이브러리를 활용했다. `leader-line`은 특정 DOM 문법을 통해 특정 html 요소들 간 연결을 다양한 css를 통해 표현할 수 있는 유용한 라이브러리이다.** 

✓ **해당 라이브러리를 활용하려면 간단한 예시로 아래와 같이 코드를 작성해야 한다.**

```jsx
import LeaderLine from 'leader-line-new' // leader-line의 새로운 버전 (이전 버전은 타입스크립트에서 아예 못읽어온다)

new LeaderLine({
	start: document.getElementById('a1'),
	end: document.getElementById('a3')
})
```

> 위와 같이 코드를 작성하면 a1이라는 id를 가진 태그와 a2라는 id를 가진 태그를 기본적은 선으로 연결이 가능하고 디테일한 css도 `{}` 내부 옵션으로 설정 가능하다.
> 

> 토폴로지 특성상 요소 간 연결들이 방대해질 수 있는 점을 생각하여 line을 만들어주는 메서드를 따로 만들어서 line을 생성해야 할 때마다 해당 메서드를 사용했다.
> 

→ 아래는 line 생성 메서드와 해당 메서드를 활용한 vue코드 그리고 그에 따른 브라우저 일부분 캡쳐본이다.

```jsx
import LeaderLine from 'leader-line-new';

import type { PlugType, PathType, SocketType } from 'leader-line-new/types/index';

export const createLine = (
  startId: string,
  endId: string,
  color: string,
  size: number,
  startSocket: SocketType,
  endSocket: SocketType,
  startPlug?: PlugType,
  endPlug?: PlugType,
  path: PathType
) => {
  return new LeaderLine({
    start: document.getElementById(`${startId}`),
    end: document.getElementById(`${endId}`),
    color,
    size,
    startSocket: startSocket,
    endSocket: endSocket,
    startPlug,
    endPlug,
    path,
  });
};
```

```jsx
// 실제 FE 코드
createLine('타원_10-2', '타원_13-2', '#000000', 2, 'bottom', 'top', 'arrow1', 'arrow1', 'grid');
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/793761ee-1da2-4c3c-8209-ce8f1147ec2d/56018029-be7e-4961-8f61-c4bb27ec866a/Untitled.png)