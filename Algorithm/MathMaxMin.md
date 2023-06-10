# 최댓값과 최솟값

### **문제 설명**

문자열 s에는 공백으로 구분된 숫자들이 저장되어 있습니다. str에 나타나는 숫자 중 최소값과 최대값을 찾아 이를 "(최소값) (최대값)"형태의 문자열을 반환하는 함수, solution을 완성하세요.

예를들어 s가 "1 2 3 4"라면 "1 4"를 리턴하고, "-1 -2 -3 -4"라면 "-4 -1"을 리턴하면 됩니다.

### 제한 조건

- s에는 둘 이상의 정수가 공백으로 구분되어 있습니다.

### 입출력 예

| s | return |
| --- | --- |
| "1 2 3 4" | "1 4" |
| "-1 -2 -3 -4" | "-4 -1" |
| "-1 -1" | "-1 -1" |

---

## 나의 코드

```jsx
function solution(s) {
  // 문자열 s - 공백으로 구분된 숫자들 저장
  // str에 나타나는 숫자 중 최솟값 & 최댓값을 찾아 "(최솟값) (최댓값)" 반환하는 함수

  // 1. s를 먼저 공백으로 구분한다
  const oArray = s.split(" ");
  const sArray = oArray.slice(0);

  // 2. 여기서 크기를 비교하여 오름차순으로 변경 - 버블정렬 사용
  for (let i = 0; i < sArray.length - 1; i++) {
    for (let j = 0; j < sArray.length - 1 - i; j++) {
      if (sArray[j] > 0 && sArray[j + 1] < 0) {
        [sArray[j], sArray[j + 1]] = [sArray[j + 1], sArray[j]];
      }
    }
    if (JSON.stringify(oArray) === JSON.stringify(sArray)) break;
  }

  // 3. 오름차순으로 변경한 sArray에서 최소값, 최대값만 뽑아서 즉, 0, 3번째에 있는 요소만 filter 처리
  const lowest = sArray[0];
  const highest = sArray[3];
  const result = `${lowest} ${highest}`;
  return result;
}
```

> 문제점 - 음수의 경우 해당 알고리즘을 인식하지 못함
> 

결국 리서치해보니 Math.max(), Math.min()이라는 메서드가 있었고 그걸 활용했다.

ex) Math.max(1, 4, -1) 의 결과는 4이다.

```jsx
function solution(s) {
  s = s.split(" "); // 배열로 변환
  let max = Math.max(...s); // Math.max()라는 메서드 사용
  let min = Math.min(...s); // 최솟값
  // console.log(min + " " + max);
  return min + " " + max;
}
```
