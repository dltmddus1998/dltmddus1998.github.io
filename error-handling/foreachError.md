# forEach - async/await 관련 에러

**서비스 코드**

```tsx
async getNACLInfo() {
  let naclInfoArr: Object[] = [];
  const vpcsArr = this.getVPCInfo();
  return (await vpcsArr).forEach(async (vpc: any) => {
    // console.log(vpc.VpcId);
    const vpcId = vpc.VpcId;

    const command = new DescribeNetworkAclsCommand({
      Filters: [
        {
          Name: 'vpc-id',
          Values: [vpcId],
        },
      ],
    });
    try {
      const response = await this.ec2Client.send(command);
      // console.log(response.NetworkAcls);
      response.NetworkAcls.forEach((networkAcl: any) => {
        const naclEntries = networkAcl.Entries.filter(
          (entry: any) =>
            entry.Egress === false &&
            entry.RuleNumber >= 100 &&
            entry.RuleNumber <= 32766,
        );
        naclInfoArr.push(naclEntries);
      });
    } catch (error) {
      console.error(`error = `, error);
    }
  });
  return naclInfoArr;
}
```

**컨트롤러 코드**

```tsx
@Get('NACL-info')
async getNACLInfo() {
  const report = await this.awsService.getNACLInfo();
  console.log(report);
  return { NACL_info: report };
}
```

위 API를 실행하였을 때, 결과값으로 빈 배열이 나오는 문제가 있다. 문제의 이유와 해결책을 찾아보자.

위 코드의 문제는 forEach 메서드가 async/await와 함께 사용될 때 기대한 대로 작동하지 않기 때문이다. forEach는 루프 내부의 프로미스가 해결되기를 기다리지 않고 예상한 값 대신에 undefined를 반환한다. (그래서 controller코드에서 반환할 값을 콘솔을 찍어보면 undefined가 나온 것이다.)

이 문제를 해결하려면 forEach 대신에 map 메서드를 사용하고 결과로 나온 프로미스 배열에 대해 await를 사용해야 한다. 이렇게 하면 루프 내부의 모든 프로미스가 해결될 때까지 기다린 다음 최종 결과를 반환한다.

```tsx
async getNACLInfo() {
  let naclInfoArr: Object[] = [];
  const vpcsArr = await this.getVPCInfo();

  await Promise.all(
    vpcsArr.map(async (vpc: any) => {
      const vpcId = vpc.VpcId;
      const command = new DescribeNetworkAclsCommand({
        Filters: [
          {
            Name: 'vpc-id',
            Values: [vpcId],
          },
        ],
      });
      try {
        const response = await this.ec2Client.send(command);
        response.NetworkAcls.forEach((networkAcl: any) => {
          const naclEntries = networkAcl.Entries.filter(
            (entry: any) =>
              entry.Egress === false &&
              entry.RuleNumber >= 100 &&
              entry.RuleNumber <= 32766,
          );
          naclInfoArr.push(naclEntries);
        });
      } catch (error) {
        console.error(`error = `, error);
      }
    }),
  );
  return naclInfoArr;
}
```

위와 같이 코드를 수정하면 다음과 같이 컨트롤러에서 올바른 값이 리턴된다.

```json
{
  "NACL_info": [
      [
          {
              "CidrBlock": "0.0.0.0/0",
              "Egress": false,
              "Protocol": "-1",
              "RuleAction": "allow",
              "RuleNumber": 100
          }
      ],
      .
			.
			.
      [
          {
              "CidrBlock": "0.0.0.0/0",
              "Egress": false,
              "Protocol": "-1",
              "RuleAction": "allow",
              "RuleNumber": 100
          }
      ]
  ]
}
```
