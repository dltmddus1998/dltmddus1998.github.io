# JadenCase 문자열 만들기

## Case

### **문제 설명**

JadenCase란 모든 단어의 첫 문자가 대문자이고, 그 외의 알파벳은 소문자인 문자열입니다. 단, 첫 문자가 알파벳이 아닐 때에는 이어지는 알파벳은 소문자로 쓰면 됩니다. (첫 번째 입출력 예 참고)

문자열 s가 주어졌을 때, s를 JadenCase로 바꾼 문자열을 리턴하는 함수, solution을 완성해주세요.

### 제한 조건

- s는 길이 1 이상 200 이하인 문자열입니다.
- s는 알파벳과 숫자, 공백문자(" ")로 이루어져 있습니다.
    - 숫자는 단어의 첫 문자로만 나옵니다.
    - 숫자로만 이루어진 단어는 없습니다.
    - 공백문자가 연속해서 나올 수 있습니다.

### 입출력 예

| s | return |
| --- | --- |
| "3people unFollowed me" | "3people Unfollowed Me" |
| "for the last week" | "For The Last Week" |

---

## 내 답변

```jsx
function solution(s) {
  // 일단 전부 소문자로 변경 후
  // 각 단어의 첫번째 문자만 대문자로 변경
  const toLowerCase = s.toLowerCase();

  const oArray = toLowerCase.split(" ");
  const sArray = oArray.slice(0);

  let result = "";
  sArray.forEach((e, i) => {
    // console.log(e.split(""));
    // 각 글자에서 첫번째 문자를 뽑아 대문자로 변경하기 위해 각 글자를 배열로 변환
    e.split("").forEach((el, idx) => {
      // idx === 0인 경우는 각 글자의 첫 문자이기 때문에 문자일 경우 대문자로 변환한다.
      if (idx === 0 && typeof el === "string") {
        el = el.toUpperCase();
      }
      if (idx === 0 && i !== 0) {
        result += " " + el;
      } else {
        result += el;
      }
    });
  });
  //   console.log(result);
  return result;
}
```

> 우선 케이스는 모두 통과하지만, 채점하면 44점이 나왔다. 여기에 무슨 문제가 있는지 확인은 못했지만 시간복잡도 관련 문제인 것 같아서 다시 생각해보았다.
> 

## 채점 후 다시 푼 내 답변 & 정답

> 생각해보니 굳이 하나씩 찾아서 대문자 소문자로 바꿔줄게 아니라 일단 모두 소문자로 변환시켜준 후, 각 단어의 첫 문자에 해당하는 것만 대문자로 변환하면 되는 것이었다.
> 

> 또한 `if (idx === 0 && i !== 0)` 라는 조건을 붙여서 빈 문자열에 단어를 하나씩 집어 넣은 부분이 시간복잡도에 영향을 준 것 같다. (실제로 이 부분 코드를 작성하면서 이건 아니다 라는 생각이 들었고 역시나 채점에서 걸림돌이 됐기때문에 배열 관련 메서드들을 하나씩 서치해보았다.)
→ *배열 내의 모든 요소 각각에 대하여 주어진 함수를 호출한 결과를 모아 새로운 배열을 반환*하는 특징을 가진 `map()` 메서드를 통해 충분히 대체할 수 있는 부분이었던 것이다.
> 

```jsx
function solution(s) {
  // 모두 소문자로 변환한 후 각 단어의 첫 문자만 대문자로 변환

  /**
   * map() 정의: 배열 내의 모든 요소 각각에 대하여 주어진 함수를 호출한 결과를 모아 새로운 배열을 반환합니다
   * -> 모두 소문자로 변환해놓은 s를
   * 배열로 변환한 후 sArray라는 변수에 할당한다. 1️⃣
   * 이후,  map을 통해 조건에 충족하는 배열을 반환한 후 2️⃣
   * join 메서드를 통해 문자열로 재반환해준다. 3️⃣
   */

	// s = "I aM gOod"
  s = s.toLowerCase(); // i am good
  const sArray = s.split(" "); // 1️⃣ ["i", "am", "good"]
  const answer = sArray
    .map((el) => {
      const elArr = el.split(""); // elArr = ["i"] ["a", "m"] ["g", "o", "o", "d"]
      if (elArr[0]) {
        elArr[0] = elArr[0].toUpperCase(); // ["I"] ["A", "m"] ["G", "o", "o", "d"]
        return elArr.join(""); // 각 문자들을 단어로 합쳐준다. 2️⃣ ["I", "Am", "Good"]
      }
    })
    .join(" "); // 3️⃣ I Am Good
  return answer;
}
```