# ๐ง ๋ฌ๋ ฅ์์ ์ฐ์ 15์ผ๊ฐ์ ๋ฐ์ดํฐ๋ง ๋ฝ์์ ๋ณด์ฌ์ฃผ๋ API๋ฅผ ๋ง๋ค์ด๋ณด์

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

  await this.db // 1๏ธโฃ
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
  await this.db // 2๏ธโฃ
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

> [Mongo Shell์ ํตํด ๋๋ฏธ๋ฐ์ดํฐ ์์ฑํ๊ธฐ (๋๋คํ ๊ฐ ๋ฃ๊ธฐ)](https://dltmddus1998.github.io/DevLog/mongoshellDummy.html)์ถ์ถํด๋์ daily ๋ฐ์ดํฐ๋ฅผ ํ๋ก ํธ์์ ๋ณด์ฌ์ฃผ๊ธฐ ์ํด ๊ด๋ จ API๋ฅผ ๋ง๋ค์ด๋ณด๋ ค๊ณ  ํ๋ค.
> 

> ์ฐ์ , ํ๋ก ํธ์์ ์ด๋ค์์ผ๋ก ๋ฐ์ดํฐ๋ฅผ ๋ณด์ฌ์ฃผ๋์ง์ ๊ดํด์ ๋งํด๋ณด๋ฉด, ์ ์ ๊ฐ ๋ฌ๋ ฅ์์ ํน์  ๋ ์ง๋ฅผ ํด๋ฆญํ๋ฉด, ํด๋น ๋ ์ง๋ก๋ถํฐ 14์ผ ํ์ ๋ฐ์ดํฐ๊น์ง, ์ฆ ์ด 15์ผ์น์ ๋ฐ์ดํฐ๋ฅผ ๊ทธ๋ํ๋ฅผ ํตํด ๋ณด์ฌ์ค ์์ ์ด๋ค.
> 

## Dayjs๋ฅผ ์ฌ์ฉํด๋ณด์

> ๋ ์ง ๊ณ์ฐ ๊ฐ์ ๊ฒฝ์ฐ ์ฒ์์๋ ์์  ํ๋ํ๊ฒ `new Date()` ๋ฅผ ์ด์ฉํ์ฌ ๊ณ์ฐํ๋๋ฐ ์ ๋๋ก๋ ๊ณ์ฐ์ด ์ด๋ ค์ ๋ค. (์๋ฒ๋ฅผ ๋๋ฆด ๋๋ง๋ค ๊ฐ์ด ๋ค๋ฅด๊ฒ ๋์จ๋ค๊ฑฐ๋, param๊ฐ์ผ๋ก ๋ค์ด๊ฐ date ๊ฐ์ด ๋ฐ๋๋ฉด 15๊ฐ๊ฐ ์๋ 300๊ฐ์ฉ ๋ฆฌํด๋๋ค๊ฑฐ๋์ ๊ฐ์ ๋๋ฌด ์๊ฐ์ง๋ ๋ชปํ ์ค๋ฅ ์ํฉ์ด ๋์๋ค.)
> 

> ์ด๋ฐ๊ฑธ๋ก ์ด๋ ค์์ ๊ฒช๊ณ  ์๋๊ฑธ ๋ณธ ํ์์ด ๊ทธ๋ ๊ฒ ํ์ง ๋ง๋ผ๊ณ  ํ๋๋ผโฆ dayjs๋ผ๋ ๋ผ์ด๋ธ๋ฌ๋ฆฌ ์ฐ๋ฉด ์์ฝ๊ฒ ํ  ์ ์๋ค๊ณ  ํ์๋ฉด์ ์ถ์ฒํด์ฃผ์จ๋๋ฐ 1์๊ฐ๋์ ๋๋๋๋ ๋ฌธ์ ๊ฐ 1๋ถ๋ง์ ํด๊ฒฐ๋ผ๋ฒ๋ ธ๋ค.๐ฅฒ
> 

๐ฑ [dayjs ์ฐธ๊ณ  ๋ฌธ์](https://day.js.org/docs/en/manipulate/add)

![image](https://user-images.githubusercontent.com/73332608/207573339-5f45c42d-6c16-4fd8-92af-294774aedb31.png)

> ์ด๋ฅผ ์ํด์๋, mongodb์ ์ ์ฅํด๋์ ์ ๋ฐ์ดํฐ์ ๊ฐ์ด timestamp๋ก ํํ๋์ด ์๋ ๋ ์ง ๋ฐ์ดํฐ๋ฅผ ํตํด ํด๋น ๊ณผ์ ์ ๊ตฌํํ  ์ ์๋ค.
> 

> 1๏ธโฃย  ์์ฒญ์ผ๋ก ๋ค์ด์จ ๋ ์ง์ ๋ง๋ ๋ฐ์ดํฐ๋ฅผ ํด๋น์ปฌ๋ ์์์ ์ฐพ์ timestamp๋ผ๋ ์ปฌ๋ผ์ 15์ผ ํ์ ๋ ์ง๋ฅผ ๊ตฌํ์ฌ after15days๋ผ๋ ๋ณ์์ ํ ๋นํ์๋ค.
> 

> 2๏ธโฃย ํด๋น ์ปฌ๋ ์์์ 1๋์น ๋ฐ์ดํฐ์ ๊ดํ์ฌ forEach๋ฅผ ํตํด ๋ฐ๋ณต๋ฌธ์ ๋๋ฉด์ ํ๋ก ํธ๋ก๋ถํฐ ์์ฒญ์จ ๋ ์ง (๋ณ์ `today`)์ `today`์์ 15์ผ ํ์ ๋ ์ง ์ฌ์ด์ ์๋ ๋ ์ง๋ค์ ํด๋นํ๋ ๋ฐ์ดํฐ๋ค์ ๋น ๋ฐฐ์ด์ pushํ๋ค.
> 

โ๏ธ **์ ์ฝ๋๋  service ๋ถ๋ถ์ ํด๋นํ๋ ์ฝ๋์ด๊ณ  controller์ module๊น์ง ๋ชจ๋ ๊ตฌ์ฑํ ํ (์ด ๋ถ๋ถ์ ์๋ตํ๊ฒ ๋ค.) postman์ผ๋ก (date=2022/2/22์ผ๋ก param๊ฐ์ ๋ณด๋ธ ํ) return ๊ฐ์ ํ์ธํด๋ณด๋ ๋ค์๊ณผ ๊ฐ์๋ค.**

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
