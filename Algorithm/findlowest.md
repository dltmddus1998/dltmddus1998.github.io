### **문제 설명**

길이가 같은 배열 A, B 두개가 있습니다. 각 배열은 자연수로 이루어져 있습니다.

배열 A, B에서 각각 한 개의 숫자를 뽑아 두 수를 곱합니다. 이러한 과정을 배열의 길이만큼 반복하며, 두 수를 곱한 값을 누적하여 더합니다. 이때 최종적으로 누적된 값이 최소가 되도록 만드는 것이 목표입니다. (단, 각 배열에서 k번째 숫자를 뽑았다면 다음에 k번째 숫자는 다시 뽑을 수 없습니다.)

예를 들어 A = `[1, 4, 2]` , B = `[5, 4, 4]` 라면

- A에서 첫번째 숫자인 1, B에서 첫번째 숫자인 5를 뽑아 곱하여 더합니다. (누적된 값 : 0 + 5(1x5) = 5)
- A에서 두번째 숫자인 4, B에서 세번째 숫자인 4를 뽑아 곱하여 더합니다. (누적된 값 : 5 + 16(4x4) = 21)
- A에서 세번째 숫자인 2, B에서 두번째 숫자인 4를 뽑아 곱하여 더합니다. (누적된 값 : 21 + 8(2x4) = 29)

즉, 이 경우가 최소가 되므로 **29**를 return 합니다.

배열 A, B가 주어질 때 최종적으로 누적된 최솟값을 return 하는 solution 함수를 완성해 주세요.

### 제한사항

- 배열 A, B의 크기 : 1,000 이하의 자연수
- 배열 A, B의 원소의 크기 : 1,000 이하의 자연수

### 입출력 예

| A | B | answer |
| --- | --- | --- |
| [1, 4, 2] | [5, 4, 4] | 29 |
| [1,2] | [3,4] | 10 |

### 입출력 예 설명

입출력 예 #1

문제의 예시와 같습니다.

입출력 예 #2

A에서 첫번째 숫자인 1, B에서 두번째 숫자인 4를 뽑아 곱하여 더합니다. (누적된 값 : 4) 다음, A에서 두번째 숫자인 2, B에서 첫번째 숫자인 3을 뽑아 곱하여 더합니다. (누적된 값 : 4 + 6 = 10)

이 경우가 최소이므로 10을 return 합니다.

---

### 나의 코드 및 정담

```jsx
function solution(A, B) {
  // 배열 A, B 두 개
  // 각각 한 개의 숫자를 뽑아 두 수 곱하기 (배열의 길이만큼 반복하여 두 수를 곱한 값 누적 합계 구하기)
  // 해당 누적값이 최소가 되도록 만드는 것이 목표

  // 어떤 경우에 최솟값이 될 수 있느냐!!
  // A에서 최댓값 찾아서 B의 최솟값과 곱해주고
  // 반대로도 진행한 후,
  // 이걸 반복하는 것.

  // 우선 크기 순서대로 배열을 sorting해주고
  // 먼저 곱한 애들은 빼주자

  //   sliceA = sliceA.sort((a, b) => a - b);
  //   sliceB = sliceB.sort((a, b) => a - b);

  // A: 오름차순 정렬 B: 내림차순 정렬
  const sliceA = A.sort((a, b) => a - b).slice();
  const sliceB = B.sort((a, b) => b - a).slice();
  let acc = 0;

  // sliceA.forEach((aData, i) => {
  //   sliceB.forEach((bData, j) => {
  //     if (i === j) acc += aData * bData;
  //   });
  // });
  sliceA.forEach((aData, i) => {
    acc += aData * sliceB[i];
  });
  return acc;
}

const A = [1, 2];
const B = [3, 4];

solution(A, B);
```
