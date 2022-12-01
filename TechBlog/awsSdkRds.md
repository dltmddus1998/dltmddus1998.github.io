# aws-sdk를 이용하여 rds 데이터 다루기

## @aws-sdk/client-rds 사용 예시

```tsx
// ⭐️ @aws-sdk/client-rds Example
  const client = new RDSClient({ region: 'ap-northeast-2' });

  const params = {
    /** input parameters */
    DBClusterIdentifier: '<cluster명>',
    RoleArn: 'arn:aws:iam::<account>:role/rds-monitoring-role',
    FeatureName: '', // option
  };
  const command = new AddRoleToDBClusterCommand(params);
  try {
    const data = await client.send(command);
    return data;
  } catch (err) {
    console.error(err);
  }
}
```

> 위와 같이 AddRoleToDBClusterCommand 메서드의 params에 해당하는 옵션들을 params에 할당해준다.
> 

> node_modules를 통해 어떤 옵션들을 써야하는지 확인할 수 있다.
> 

```tsx
DBClusterIdentifier: string | undefined;

RoleArn: string | undefined;

FeatureName?: string;
```

> DBClusterIdentifier은 말 그대로 DB 클러스터명, RoleArn은 해당 역할을 부여할 IAM 계정의 ARN을 의미한다.
> 

✋🏻 **맨 위에 있는 코드(service.ts)를 컨트롤러(controller.ts) 코드를 통해 실행시키면 아래에 대한 결과값이 나오는 것을 확인할 수 있다.**

```json
httpStatusCode: 200,
requestId: '16c8ac49-6ccc-4aac-9fe3-c59260cd04df',
attempts: 1,
totalRet
```

<aside>
⚠️ 그렇지만 아직 이 결과값의 데이터가 어떤식으로 가공되어야 하고 어떻게 활용할 수 있을지는 파악하지 못했기 때문에 뒤에 client-rds-data에 대한 실습까지 마무리 해놓고 이 부분에 대해 리서치해보려 한다.

</aside>

## @aws-sdk/client-rds-data 사용 예시

> @aws-sdk/client-rds-data에서 AddTagsToResourceCommand() 메서드를 사용해보자.
→ 해당 메서드는 RDS에서 사용되는 태그를 추가해주는 간단한 메서드이다.
> 

💡 **여기에서 쓰이는 옵션들을 보면 다음과 같다.**

```tsx
port interface Tag {
	Key?: string;

	Value? string;
}

ResourceName: string | undefined;

Tags: Tag[] : undefined;
```

> ResourceName은 사용하고자 하는 RDS의 구분 ARN을 의미한다. 
(AWS 사이트에서 RDS Dashboard를 통해 configuration 탭에서 알 수 있다.)
> 

> Tags는 추가하고자 하는 태그의 정보를 입력하면 되는데 다로 정의된 Tag타입으로 해당 값을 입력하면 된다.
> 

✋🏻 **위의 옵션 정보들을 활용하여 다음과 같이 코드를 짜보았다.**

```tsx
const tagParams = {
  ResourceName: 'arn:aws:rds:ap-northeast-2:<account>:cluster:<cluster명>',
  Tags: [{ Key: 'cde', Value: '333' }],
};
const result = new AwsRds.AddTagsToResourceCommand(tagParams);
try {
  const data = await client.send(result);
  return data;
} catch (err) {
  console.error(err);
}
```

> tagParams를 통해 옵션들을 설정하여 result 값을 리턴하면 위에서 리턴했을 때처럼 나오는데, httpStatusCode가 200이면서 다음과 같이 나오면 메서드가 제대로 실행됐다는 것을 의미하는 것 같다.
> 

```json
{
    "$metadata": {
        "httpStatusCode": 200,
        "requestId": "f31df5b8-6ddf-44a8-8ffe-78efd6719f95",
        "attempts": 1,
        "totalRetryDelay": 0
    }
}
```

💡 **그리고 다음과 같이 태그가 정상적으로 추가됐음을 알 수 있다.**

![image](https://user-images.githubusercontent.com/73332608/204968755-43d31935-9874-4611-8d45-8e35dbc59286.png)
