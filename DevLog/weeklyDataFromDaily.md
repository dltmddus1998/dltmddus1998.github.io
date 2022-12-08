# Json 파일의 데이터 가공 (JavaScript) [부제: reduce]

> [Mongo Shell을 통해 더미데이터 생성하기 (랜덤한 값 넣기)](https://dltmddus1998.github.io/DevLog/mongoshellDummy.html)에서 daily 데이터를 1년치를 추출했는데 이 데이터를 바탕으로 평균치를 내서 weekly 데이터를 추출하려했다.
> 

> 처음에는 단순히 forEach 함수를 활용해서 Json파일에서 depth가 깊은 값을 가져오려 했는데 데이터 양이 방대하기도 하고 depth가 깊어 복잡했기 때문에 잘 되지 않았다.
> 

> 그래서 아래와 같이 reduce를 활용했다.
> 

```jsx
const main = async () => {
  const res = await fetch('http://localhost:3000/data');
  const json = await res.json();

  const data = json.slice(-7);

  const division = (arr, n) => {
    const length = arr.length;
    const divide = Math.floor(length / n) + (Math.floor(length % n) > 0 ? 1 : 0);
    const newArray = [];
    for (let i = 0; i <= divide; i++) {
      newArray.push(arr.splice(0, n));
    }
    return newArray;
  };
  const dividedData = division(json, 7);
  let arr = [];
  let result;
  dividedData.forEach((weeklyData) => {
    if (weeklyData.length) {
      result = weeklyData.reduce((a, c) => {
        c.dailyEC2Stats.forEach((eachData, idx) => {
          if (eachData.region in a) {
            a[eachData.region] = {
              countTotalEC2: a[eachData.region].countTotalEC2 + eachData.countTotalEC2,
              countUnmanagedEC2: a[eachData.region].countUnmanagedEC2 + eachData.countUnmanagedEC2,
              countUnallowedSpec: a[eachData.region].countUnallowedSpec + eachData.countUnallowedSpec,
            };
          } else { // 1️⃣
            a[eachData.region] = {
              countTotalEC2: eachData.countTotalEC2,
              countUnmanagedEC2: eachData.countUnmanagedEC2,
              countUnallowedSpec: eachData.countUnallowedSpec,
            };
          }
        });
        return a;
      }, {});
      arr.push(result);
    }
  });

  let resultArr = [];
  arr.forEach((data) => {
    const obj = Object.entries(data).map(([key, value]) => ({
      region: key,
      AvgOfCountTotalEC2: Math.floor(value.countTotalEC2 / 7),
      AvgOfCountUnmanagedEC2: Math.floor(value.countUnmanagedEC2 / 7),
      AvgOfCountUnallowedSpec: Math.floor(value.countUnallowedSpec / 7),
    }));
    resultArr.push(obj);
  });
  console.log(resultArr);
};

main();
```

> a, 즉 누산기에는 객체 형태로 리전별 daily 데이터들의 합산 값을 할당할 수 있도록 1️⃣ 과 같이 구현했다.
> 
> 리턴된 a는 주간마다 (7일마다) 반복되는 값이므로 빈 배열에 넣어주었다.
>
> reduce 내에서 누산기 값은 반복적으로 변하기 때문에 평균값을 위해 나누기를 하면 값이 이상하게 나온다.
>
> 이를 방지하기 위해 따로 `Object.entries()`를 활용하여 객체의 모양을 다듬어줌과 동시에 평균값을 넣어줬다.
> 

✔️ **위 코드를 통해 Json 파일을 추출하면 다음과 같이 나온다.**

```json
[
...
	[
	  {
	    "region": "ap-northeast-2",
	    "AvgOfCountTotalEC2": 72,
	    "AvgOfCountUnmanagedEC2": 32,
	    "AvgOfCountUnallowedSpec": 4
	  },
	  {
	    "region": "us-east-1",
	    "AvgOfCountTotalEC2": 93,
	    "AvgOfCountUnmanagedEC2": 30,
	    "AvgOfCountUnallowedSpec": 6
	  },
	  {
	    "region": "ap-northeast-3",
	    "AvgOfCountTotalEC2": 105,
	    "AvgOfCountUnmanagedEC2": 23,
	    "AvgOfCountUnallowedSpec": 4
	  },
	  {
	    "region": "ap-northeast-1",
	    "AvgOfCountTotalEC2": 115,
	    "AvgOfCountUnmanagedEC2": 54,
	    "AvgOfCountUnallowedSpec": 5
	  },
	  {
	    "region": "sa-east-1",
	    "AvgOfCountTotalEC2": 116,
	    "AvgOfCountUnmanagedEC2": 49,
	    "AvgOfCountUnallowedSpec": 4
	  }
	]
...
]
```
