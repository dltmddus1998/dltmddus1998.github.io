# 🧐 달력에서 연속 15일간의 데이터만 뽑아서 보여주는 API를 만들어

```tsx
public async findDailyTrend({
    date,
}: FindDailyTrendRequestDto): Promise<SvcResponse> {
  const requestDate = new Date(
    `${date.split('.')[0]}/${Number(date.split('.')[1])}/${Number(
      date.split('.')[2]
    )}`
  );

  const today = dayjs(requestDate).format();
  let after15days;

  await this.db // 1️⃣
    .collection(`[COLLECTION NAME]`)
    .aggregate([])
    .toArray()
    .then((allDailyDatasArr: any) => {
      allDailyDatasArr.forEach((dailyDatas: any) => {
        const date = new Date(dailyDatas.datas.timestamp).toLocaleString();
        if (date === requestDate.toLocaleString()) {
          after15days = dayjs(requestDate).add(15, 'day').format();
        }
      });
    });

  let resultArr = [];
  await this.db // 2️⃣
    .collection(`[COLLECTION NAME]`)
    .find()
    .toArray()
    .then((allDatas) => {
      allDatas.forEach((data: any) => {
        if (
          today <= dayjs(new Date(data.datas.timestamp)).format() &&
          after15days > dayjs(new Date(data.datas.timestamp)).format()
        ) {
          resultArr.push(data.datas);
        }
      });
    });
  return {
    data: JSON.stringify(resultArr),
    statusCode: HttpStatus.OK,
  };
}
```

> [Mongo Shell을 통해 더미데이터 생성하기 (랜덤한 값 넣기)](https://dltmddus1998.github.io/DevLog/mongoshellDummy.html)추출해놓은 daily 데이터를 프론트에서 보여주기 위해 관련 API를 만들어보려고 한다.
> 

> 우선, 프론트에서 어떤식으로 데이터를 보여주는지에 관해서 말해보면, 유저가 달력에서 특정 날짜를 클릭하면, 해당 날짜로부터 14일 후의 데이터까지, 즉 총 15일치의 데이터를 그래프를 통해 보여줄 예정이다.
> 

## Dayjs를 사용해보자

> 날짜 계산 같은 경우 처음에는 완전 하드하게 `new Date()` 를 이용하여 계산했는데 제대로된 계산이 어려웠다. (서버를 돌릴 때마다 값이 다르게 나온다거나, param값으로 들어간 date 값이 바뀌면 15개가 아닌 300개씩 리턴된다거나와 같은 너무 생각지도 못한 오류 상황이 나왔다.)
> 

> 이런걸로 어려움을 겪고 있는걸 본 팀원이 그렇게 하지 말라고 하더라… dayjs라는 라이브러리 쓰면 손쉽게 할 수 있다고 하시면서 추천해주셨는데 1시간동안 낑낑대던 문제가 1분만에 해결돼버렸다.🥲
> 

🖱 [dayjs 참고 문서](https://day.js.org/docs/en/manipulate/add)

![image](https://user-images.githubusercontent.com/73332608/207573339-5f45c42d-6c16-4fd8-92af-294774aedb31.png)

> 이를 위해서는, mongodb에 저장해놓은 위 데이터와 같이 timestamp로 표현되어 있는 날짜 데이터를 통해 해당 과정을 구현할 수 있다.
> 

> 1️⃣  요청으로 들어온 날짜에 맞는 데이터를 해당컬렉션에서 찾아 timestamp라는 컬럼에 15일 후의 날짜를 구하여 after15days라는 변수에 할당하였다.
> 

> 2️⃣ 해당 컬렉션에서 1년치 데이터에 관하여 forEach를 통해 반복문을 돌면서 프론트로부터 요청온 날짜 (변수 `today`)와 `today`에서 15일 후의 날짜 사이에 있는 날짜들에 해당하는 데이터들을 빈 배열에 push했다.
> 

✔︎ **위 코드는  service 부분에 해당하는 코드이고 controller와 module까지 모두 구성한 후 (이 부분은 생략하겠다.) postman으로 (date=2022/2/22으로 param값을 보낸 후) return 값을 확인해보니 다음과 같았다.**

```json
{
	"statusCode": 200,
	"data": [
		{
	      "timestamp": "2/1/2022, 12:00:00 AM",
	      "dailyEC2Stats": [
	          {
	              "region": "ap-northeast-2",
	              "isAllowed": true,
	              "countTotalEC2": 74,
	              "countUnmanagedEC2": 65,
	              "countUnallowedSpec": 1
	          },
	          {
	              "region": "us-east-1",
	              "isAllowed": true,
	              "countTotalEC2": 89,
	              "countUnmanagedEC2": 46,
	              "countUnallowedSpec": 6
	          },
	          {
	              "region": "ap-northeast-3",
	              "isAllowed": true,
	              "countTotalEC2": 109,
	              "countUnmanagedEC2": 100,
	              "countUnallowedSpec": 0
	          },
	          {
	              "region": "ap-northeast-1",
	              "isAllowed": true,
	              "countTotalEC2": 111,
	              "countUnmanagedEC2": 104,
	              "countUnallowedSpec": 4
	          },
	          {
	              "region": "sa-east-1",
	              "isAllowed": true,
	              "countTotalEC2": 116,
	              "countUnmanagedEC2": 21,
	              "countUnallowedSpec": 6
	          }
	      ]
	  },
		.
		.
		.
		{
		    "timestamp": "2/15/2022, 12:00:00 AM",
		    "dailyEC2Stats": [
		        {
		            "region": "ap-northeast-2",
		            "isAllowed": true,
		            "countTotalEC2": 74,
		            "countUnmanagedEC2": 44,
		            "countUnallowedSpec": 1
		        },
		        {
		            "region": "us-east-1",
		            "isAllowed": true,
		            "countTotalEC2": 93,
		            "countUnmanagedEC2": 82,
		            "countUnallowedSpec": 5
		        },
		        {
		            "region": "ap-northeast-3",
		            "isAllowed": true,
		            "countTotalEC2": 102,
		            "countUnmanagedEC2": 43,
		            "countUnallowedSpec": 1
		        },
		        {
		            "region": "ap-northeast-1",
		            "isAllowed": true,
		            "countTotalEC2": 111,
		            "countUnmanagedEC2": 81,
		            "countUnallowedSpec": 2
		        },
		        {
		            "region": "sa-east-1",
		            "isAllowed": true,
		            "countTotalEC2": 107,
		            "countUnmanagedEC2": 97,
		            "countUnallowedSpec": 3
		        }
		    ]
		}
	]
}
```
