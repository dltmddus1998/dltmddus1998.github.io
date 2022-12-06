# Mongo Shell을 통해 더미데이터 생성하기 (랜덤한 값 넣기)

> mongoshell을 이용하여 진행중인 프로젝트에 필요한 dummy data를 삽입하였다.
> 

✔️ 날짜를 `2022/1/1` 부터 `2022/12/31` 까지 1년간의 데이터를 넣는 형태로 진행했고 count에 해당하는 컬럼들의 데이터는 정해진 max 변수까지의 범위 내에서 random한 수가 주어지도록 구현했다.

✔️ for문을 이용하여 365개에 해당하는 개수의 데이터들을 mongodb의 stats_ec2_datas_v4라는 컬렉션에 한번에 삽입할 수 있었다.

### Mongo shell - ec2

```jsx
// const max = [해당 리전의 ec2 최대 개수];
const max_n2 = 17;
const max_e1 = 12;
const max_n3 = 10;
const max_n1 = 5;
const max_se1 = 9;
let date = new Date(2022, 0, 0);

for (let i = 0; i < 364; i++) {
  date.setDate(date.getDate() + 1);
  db.stats_ec2_datas_v4.insertOne({
    timestamp: date.toLocaleString(),
    dailyEC2Stats: [
      {
        region: 'ap-northeast-2',
        isAllowed: true,
        countTotalEC2: max_n2,
        countUnmanagedEC2: Math.floor(Math.random() * max_n2),
        countUnallowedRegion: Math.floor(Math.random() * 10),
        countUnallowedSpec: Math.floor(Math.random() * 10),
      },
      {
        region: 'us-east-1',
        isAllowed: true,
        countTotalEC2: max_e1,
        countUnmanagedEC2: Math.floor(Math.random() * max_e1),
        countUnallowedRegion: Math.floor(Math.random() * 10),
        countUnallowedSpec: Math.floor(Math.random() * 10),
      },
      {
        region: 'ap-northeast-3',
        isAllowed: true,
        countTotalEC2: max_n3,
        countUnmanagedEC2: Math.floor(Math.random() * max_n3),
        countUnallowedRegion: Math.floor(Math.random() * 10),
        countUnallowedSpec: Math.floor(Math.random() * 10),
      },
      {
        region: 'ap-northeast-1',
        isAllowed: true,
        countTotalEC2: max_n1,
        countUnmanagedEC2: Math.floor(Math.random() * max_n1),
        countUnallowedRegion: Math.floor(Math.random() * 10),
        countUnallowedSpec: Math.floor(Math.random() * 10),
      },
      {
        region: 'sa-east-1',
        isAllowed: true,
        countTotalEC2: max_se1,
        countUnmanagedEC2: Math.floor(Math.random() * max_se1),
        countUnallowedRegion: Math.floor(Math.random() * 10),
        countUnallowedSpec: Math.floor(Math.random() * 10),
      },
    ],
  });
}
```
