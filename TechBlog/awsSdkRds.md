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


## 🧐 본격적인 활용 시작

> 위에서는 아직 aws-sdk에 대한 파악이 덜됐기 때문에 아무 메서드나 활용해보았는데, 이제 업무에 필요할만한 메서드를 찾아서 예시 코드를 짜보려고 한다.
> 

💡 **우선, 첫번째로 눈에 보였던 메서드는 `DescribeAccountAttributesCommand` 였다.**

> 해당 메서드는 특정 계정의 RDS 한도에 대한 데이터를 조회하는데 쓰인다.
> 

> 특히, DBInstances, AuthorizationsPerDBSecurityGroup 등 명칭별로 사용되고 있는 개수(Used)와 한도(Max)를 조회할 수 있다.
> 

👉🏻 **별다른 옵션 없이 호출만 하면 다음과 같이 결과값을 리턴할 수 있다.**

```json
{
  "$metadata": {
      "httpStatusCode": 200,
      "requestId": "c12f1377-f735-4e8b-8545-f4a48362692f",
      "attempts": 1,
      "totalRetryDelay": 0
  },
  "AccountQuotas": [
    {
        "AccountQuotaName": "DBInstances",
        "Used": 1,
        "Max": 40
    },
    {
        "AccountQuotaName": "ReservedDBInstances",
        "Used": 0,
        "Max": 40
    },
    {
        "AccountQuotaName": "AllocatedStorage",
        "Used": 0,
        "Max": 100000
    },
    {
        "AccountQuotaName": "DBSecurityGroups",
        "Used": 0,
        "Max": 25
    },
    {
        "AccountQuotaName": "AuthorizationsPerDBSecurityGroup",
        "Used": 0,
        "Max": 20
    },
    {
        "AccountQuotaName": "DBParameterGroups",
        "Used": 0,
        "Max": 50
    },
    {
        "AccountQuotaName": "ManualSnapshots",
        "Used": 3,
        "Max": 100
    },
    {
        "AccountQuotaName": "ManualClusterSnapshots",
        "Used": 0,
        "Max": 100
    },
    {
        "AccountQuotaName": "EventSubscriptions",
        "Used": 0,
        "Max": 20
    },
    {
        "AccountQuotaName": "DBSubnetGroups",
        "Used": 3,
        "Max": 50
    },
    {
        "AccountQuotaName": "OptionGroups",
        "Used": 0,
        "Max": 20
    },
    {
        "AccountQuotaName": "SubnetsPerDBSubnetGroup",
        "Used": 6,
        "Max": 20
    },
    {
        "AccountQuotaName": "ReadReplicasPerMaster",
        "Used": 0,
        "Max": 15
    },
    {
        "AccountQuotaName": "DBInstanceRoles",
        "Used": 0,
        "Max": 5
    },
    {
        "AccountQuotaName": "DBClusters",
        "Used": 1,
        "Max": 40
    },
    {
        "AccountQuotaName": "DBClusterParameterGroups",
        "Used": 0,
        "Max": 50
    },
    {
        "AccountQuotaName": "CustomEndpointsPerDBCluster",
        "Used": 0,
        "Max": 5
    },
    {
        "AccountQuotaName": "DBClusterRoles",
        "Used": 1,
        "Max": 5
    }
  ]
}
```

💡 **두 번째로 눈에 보였던 메서드는 `DescribeDBInstancesCommand`였다.**

> 제공된 RDS 인스턴스에 대한 정보를 조회할 수 있다. 
또한 그래프DB인 Amazon Neptune DB와 문서DB인 Amazon DocumentDB도 지원한다.
> 

> 결과값을 보면 알겠지만 해당 DB 인스턴스에 대한 거의 모든 정보들이 출력되는 것을 알 수 있다. 
아마 이 메서드를 사용하면 웬만한 데이터는 추출할 수 있을 것 같다. (결과값이 방대하고, 숨겨야 하는 값이 너무 많아서 생략하겠다.)
> 
