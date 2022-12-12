# Json 파일의 데이터 가공 (JavaScript) - 2 [Monthly Data]

> [Mongo Shell을 통해 더미데이터 생성하기 (랜덤한 값 넣기)](https://dltmddus1998.github.io/DevLog/mongoshellDummy.html) 와 [Json 파일의 데이터 가공 (JavaScript) [부제: reduce]](https://www.notion.so/Memo-07e20dbd1bf04718bdc506f83b3ac60a)를 통해 daily 데이터와 weekly 데이터를 추출했는데 이제 마지막으로 monthly 데이터를 추출하려 한다.
> 

> 방식은 weekly 데이터를 추출했을 때와 거의 동일하지만, weekly 데이터 추출시 사용했던 division 함수로 json 파일을 월단위로 쪼개기는 힘들었다. (매월 일수가 다르기 때문이다.)
> 

> 이 부분을 해결하기 위해 여러 방법을 사용해봤지만 일단 깔끔한 코드보다는 더미데이터 생성이 목적이므로 원초적인 방법을 사용했다. (이 부분은 시간이 남을 때 깔끔한 코드로 다시 리팩토링할 계획이다)
> 

```jsx
const json = await res.json();
let month;
let monthArr = [];
let jan = [];
let feb = [];
let mar = [];
let apr = [];
let may = [];
let jun = [];
let jul = [];
let aug = [];
let sep = [];
let oct = [];
let nov = [];
let dec = [];
let monthlyData = [];

// monthlyData.forEach

json.map((data, idx) => { // 1️⃣
  // 월별로 데이터 나누기
  if (data.timestamp.split('')[1] === '/') {
    month = Number(data.timestamp.split('')[0]);
  } else if (data.timestamp.split('')[1] !== '/') {
    month = Number(data.timestamp.split('')[0] + '' + data.timestamp.split('')[1]);
  }
  monthArr.push(month);

  if (month === 1) jan.push(data);
  if (month === 2) feb.push(data);
  if (month === 3) mar.push(data);
  if (month === 4) apr.push(data);
  if (month === 5) may.push(data);
  if (month === 6) jun.push(data);
  if (month === 7) jul.push(data);
  if (month === 8) aug.push(data);
  if (month === 9) sep.push(data);
  if (month === 10) oct.push(data);
  if (month === 11) nov.push(data);
  if (month === 12) dec.push(data);
});
monthlyData.push(jan);
monthlyData.push(feb);
monthlyData.push(mar);
monthlyData.push(apr);
monthlyData.push(may);
monthlyData.push(jun);
monthlyData.push(jul);
monthlyData.push(aug);
monthlyData.push(sep);
monthlyData.push(oct);
monthlyData.push(nov);
monthlyData.push(dec);
```

> 1️⃣ 은 {timestamp: 2/21/2022, 12:00:00 AM} 와 같이 되어 있는 시간 데이터에서 split을 통해 몇월인지 알아오기 위한 코드이다.
> 

> 일단 위와 같이 매달 따로 배열에 담아준 후 2차원 배열 형태로 큰 배열에 다시 한번 push하는 형태로 만들었다.
> 

```jsx
let arr = [];
let result;

monthlyData.forEach((monthlyData) => {
  if (monthlyData.length) {
    result = monthlyData.reduce((a, c) => {
      c.dailyEC2Stats.forEach((eachData) => {
        if (eachData.region in a) {
          a[eachData.region] = {
            countTotalEC2: a[eachData.region].countTotalEC2 + eachData.countTotalEC2,
            countUnmanagedEC2: a[eachData.region].countUnmanagedEC2 + eachData.countUnmanagedEC2,
            countUnallowedSpec: a[eachData.region].countUnallowedSpec + eachData.countUnallowedSpec,
          };
        } else {
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
arr.forEach((data, idx) => {
  const obj = Object.entries(data).map(([key, value]) => {
    // 1월, 3월, 5월, 7월, 8월, 10월, 12월
    if (idx === 0 || idx === 2 || idx === 4 || idx === 6 || idx === 7 || idx === 9 || idx === 11) {
      return {
        region: key,
        AvgOfCountTotalEC2: Math.floor(value.countTotalEC2 / 31),
        AvgOfCountUnmanagedEC2: Math.floor(value.countUnmanagedEC2 / 31),
        AvgOfCountUnallowedSpec: Math.floor(value.countUnallowedSpec / 31),
      };
    } else if (idx === 1) {
      // 2월
      return {
        region: key,
        AvgOfCountTotalEC2: Math.floor(value.countTotalEC2 / 28),
        AvgOfCountUnmanagedEC2: Math.floor(value.countUnmanagedEC2 / 28),
        AvgOfCountUnallowedSpec: Math.floor(value.countUnallowedSpec / 28),
      };
    } else {
      return {
        region: key,
        AvgOfCountTotalEC2: Math.floor(value.countTotalEC2 / 30),
        AvgOfCountUnmanagedEC2: Math.floor(value.countUnmanagedEC2 / 30),
        AvgOfCountUnallowedSpec: Math.floor(value.countUnallowedSpec / 30),
      };
    }
  });
  resultArr.push(obj);
});

resultArr.map((result, idx) => { // 2️⃣
  result.unshift({ month: `${idx + 1}월` });
});
const file = 'monthly_stats_ec2_datas_v1.json';
fs.writeFile(file, JSON.stringify(resultArr), (err) => {
  if (err) console.error(err);
});
```

> weekly 데이터 때와 동일하게 reduce를 통해 누적값으로 합을 구한 후 Object.entries()를 활용하여 객체 모양을 다듬어주면서 평균값을 주었는데 위에서 보여준바와 같이 31일인 달, 2월, 30일인 달 이 세 경우를 모두 나눠서 평균값을 내주었다.
> 

> 이후 몇월인지 좀더 명시적으로 표시해주기 위해 2️⃣와 같이 구현했다.
> 

✔️ **위 코드를 통해 Json 파일을 추출하면 다음과 같이 나온다.**

```json
[
	[
    { "month": "1월" },
    { "region": "ap-northeast-2", "AvgOfCountTotalEC2": 64, "AvgOfCountUnmanagedEC2": 33, "AvgOfCountUnallowedSpec": 3 },
    { "region": "us-east-1", "AvgOfCountTotalEC2": 85, "AvgOfCountUnmanagedEC2": 42, "AvgOfCountUnallowedSpec": 4 },
    { "region": "ap-northeast-3", "AvgOfCountTotalEC2": 98, "AvgOfCountUnmanagedEC2": 49, "AvgOfCountUnallowedSpec": 5 },
    { "region": "ap-northeast-1", "AvgOfCountTotalEC2": 106, "AvgOfCountUnmanagedEC2": 51, "AvgOfCountUnallowedSpec": 5 },
    { "region": "sa-east-1", "AvgOfCountTotalEC2": 108, "AvgOfCountUnmanagedEC2": 41, "AvgOfCountUnallowedSpec": 4 }
  ],
	.
	.
	.
	[
    { "month": "12월" },
    { "region": "ap-northeast-2", "AvgOfCountTotalEC2": 70, "AvgOfCountUnmanagedEC2": 32, "AvgOfCountUnallowedSpec": 3 },
    { "region": "us-east-1", "AvgOfCountTotalEC2": 92, "AvgOfCountUnmanagedEC2": 47, "AvgOfCountUnallowedSpec": 6 },
    { "region": "ap-northeast-3", "AvgOfCountTotalEC2": 104, "AvgOfCountUnmanagedEC2": 48, "AvgOfCountUnallowedSpec": 4 },
    { "region": "ap-northeast-1", "AvgOfCountTotalEC2": 114, "AvgOfCountUnmanagedEC2": 49, "AvgOfCountUnallowedSpec": 4 },
    { "region": "sa-east-1", "AvgOfCountTotalEC2": 114, "AvgOfCountUnmanagedEC2": 55, "AvgOfCountUnallowedSpec": 4 }
  ]
]
```
