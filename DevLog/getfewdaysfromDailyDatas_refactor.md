# 달력에서 연속 15일간의 데이터만 뽑아서 보여주는 API를 만들어보자 (Ver. 2)

## 🤔 For What?

> [달력에서 연속 15일간의 데이터만 뽑아서 보여주는 API 만들기](https://dltmddus1998.github.io/DevLog/getfewdaysfromDailyDatas.html)에서 구현한 API에서 몇가지 문제점을 발견해서 리팩토링을 진행했다.
> 

## 🗑 원래 코드 👉🏻

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

  await this.db 
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
  await this.db 
    .collection(`[COLLECTION NAME]`)
    .find() // 💡
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

### 🧐 문제점

☑️ 💡 를 보면, mongoDB의 `db.collection.find()`  메서드 자체에 조건을 주는 것이 아닌 `.then()`으로 모든 데이터를 받아와서 그 안에서 if 조건문을 통해 데이터를 걸러내는 방식을 사용했다.

👉🏻 해당 방식은 현재처럼 많지 않은 데이터로만 코드를 돌릴 때는 별 문제 없이 돌아가겠지만, 추후에 development 단계나 production 단계에서는 이보다 훨씬 많은 데이터가 들어가기 때문에 클린 코드라고 말하기 힘들다.

🧐 `find()`를 통해 많은 데이터를 모두 불러온 후 조건을 주는 방식이기 때문이다.

☑️ 위에 코드를 보면 중복되는 코드가 없어 보이지만, 현재 프로젝트에서 같은 방식의 daily 데이터 추출 API가 3개 필요하다. 따라서 겹치는 코드가 많다.

### 💡 해결책

✅ `find()`에서 `{ }` 내부에 옵션 조건을 줘서 처음부터 모든 데이터를 가져오지 않도록 구현한다.

👉🏻 조건에 맞는 데이터만 가져온다.

✅ 따로 private한 메서드를 선언해서 중복된 코드를 최소화한다.

## ♻️ 리팩토링한 코드

### private 메서드 선언 - 중복된 코드 최소화

```tsx
private async find15daysData(
  date: string,
  subtractedDate: string,
  collectionName: string
) {
  const data = await this.db
    .collection(collectionName)
    .find({
      $expr: {
        $and: [
          {
            $gt: [
              {
                $dateFromString: {
                  dateString: '$datas.timestamp',
                },
              },
              new Date(subtractedDate),
            ],
          },
          {
            $lt: [
              {
                $dateFromString: {
                  dateString: '$datas.timestamp',
                },
              },
              new Date(date),
            ],
          },
        ],
      },
    })
    .toArray();

  data.sort((a: any, b: any) => {
    if (new Date(a.datas.timestamp) > new Date(b.datas.timestamp)) {
      return 1;
    } else {
      return -1;
    }
  });

  return data;
}

private convertStartDate(startDate: string): string {
  let sMonth =
    startDate.slice(4, 5) === '0' ? startDate.slice(5, 6) : startDate.slice(4, 6);
  let sDate =
    startDate.slice(6, 7) === '0' ? startDate.slice(7, 8) : startDate.slice(6, 8);
  let sYear = startDate.slice(0, 4);

  const date: string = `${sMonth}/${Number(sDate) + 1}/${sYear}`;

  return date;
}
```

### 메인 API - 코드 효율성 up (조건에 맞는 데이터만 find)

```tsx
public async findEC2DailyTrend({
    date,
  }: FindDailyTrendRequestDto): Promise<SvcResponse> {
    const pickedDate = this.convertStartDate(date);
    const before15daysDate = dayjs(pickedDate).subtract(15, 'day').format();
    console.log(pickedDate, before15daysDate);

    const data = await this.find15daysData(
      pickedDate,
      before15daysDate,
      `[collection name]`
    );

    if (data.length < 15) {
      return {
        data: JSON.stringify(data),
        message: 'under 15',
        statusCode: HttpStatus.OK,
      };
    }

    return {
      data: JSON.stringify(data),
      statusCode: HttpStatus.OK,
    };
  }
```

### 💡 코드 설명

> find15daysData와 convertStartDate를 통해 중복된 코드를 메서드로 선언해주었다.
> 

✔️ find15daysData는 find() 내부의 옵션을 통해 조건을 줘서 필요한 데이터만 가져온 data를 날짜의 내림차순으로 재정렬하여 리턴하는 메서드이다.

✔️ convertStartDate는 Query로 받아온 데이터를 MongoDB에서 인식할 수 있는 String 타입으로 변환하는 메서드이다.

👉🏻 find15daysData를 보면 알겠지만, 해당 String 타입은 new Date()를 통해 Date 타입으로 변환해주고, MongoDB의 String 타입으로 저장된 날짜를 Date 타입으로 변환해주는 $dateFromString을 통해 convertStartDate를 통해 변환한 날짜와 $gt, $lt를 통해 비교하여 조건에 해당하는 데이터들만 추출했다.

## 데이터 추출 결과

```json
{
	"data": [
		{
        "datas": {
            "timestamp": "2/19/2022, 12:00:00 AM",
            "dailyEC2Stats": [
                {
                    "region": "ap-northeast-2",
                    "isAllowed": true,
                    "countTotalEC2": 62,
                    "countUnmanagedEC2": 45,
                    "countUnallowedSpec": 9
                },
                {
                    "region": "us-east-1",
                    "isAllowed": true,
                    "countTotalEC2": 96,
                    "countUnmanagedEC2": 14,
                    "countUnallowedSpec": 1
                },
                {
                    "region": "ap-northeast-3",
                    "isAllowed": true,
                    "countTotalEC2": 109,
                    "countUnmanagedEC2": 61,
                    "countUnallowedSpec": 6
                },
                {
                    "region": "ap-northeast-1",
                    "isAllowed": true,
                    "countTotalEC2": 111,
                    "countUnmanagedEC2": 98,
                    "countUnallowedSpec": 8
                },
                {
                    "region": "sa-east-1",
                    "isAllowed": true,
                    "countTotalEC2": 120,
                    "countUnmanagedEC2": 54,
                    "countUnallowedSpec": 0
                }
            ]
        }
    },
		.
		.
		.
    {
      "datas": {
        "timestamp": "3/5/2022, 12:00:00 AM",
        "dailyEC2Stats": [
            {
                "region": "ap-northeast-2",
                "isAllowed": true,
                "countTotalEC2": 71,
                "countUnmanagedEC2": 50,
                "countUnallowedSpec": 6
            },
            {
                "region": "us-east-1",
                "isAllowed": true,
                "countTotalEC2": 96,
                "countUnmanagedEC2": 75,
                "countUnallowedSpec": 4
            },
            {
                "region": "ap-northeast-3",
                "isAllowed": true,
                "countTotalEC2": 101,
                "countUnmanagedEC2": 36,
                "countUnallowedSpec": 8
            },
            {
                "region": "ap-northeast-1",
                "isAllowed": true,
                "countTotalEC2": 110,
                "countUnmanagedEC2": 83,
                "countUnallowedSpec": 5
            },
            {
                "region": "sa-east-1",
                "isAllowed": true,
                "countTotalEC2": 118,
                "countUnmanagedEC2": 75,
                "countUnallowedSpec": 2
            }
          ]
      }
    }
	]
}
```
