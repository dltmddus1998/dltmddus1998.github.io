# 클라우드 콘솔 및 서버 관련 IAM 정보 가져오기 (aws-sdk 사용)

## 클라우드 콘솔 IAM

```tsx
import {
  IAMClient,
  GetUserCommand,
  ListAttachedUserPoliciesCommand,
  ListAccessKeysCommand,
  ListGroupsForUserCommand,
  ListAttachedGroupPoliciesCommand,
} from '@aws-sdk/client-iam';
import { fromIni } from '@aws-sdk/credential-provider-ini';

private readonly client: IAMClient;
constructor() {
	this.client = new IAMClient({
		credentials: fromIni({
			profile: process.env.xxxxx
		})
	})	
}
.
.
.

async getConsoleIAMInfo() {
    /**
     * 클라우드 콘솔 - IAM
     */
    try {
      const user = await this.client.send(new GetUserCommand({}));
      const policies = await this.client.send(
        new ListAttachedUserPoliciesCommand({
          UserName: user.User.UserName,
        }),
      ); // 사용자가 연결된 모든 IAM 정책 목록 검색
      const accessKeys = await this.client.send(
        new ListAccessKeysCommand({ UserName: user.User.UserName }),
      ); // 사용자의 모든 엑세스키 목록 검색
      const groups = await this.client.send(
        new ListGroupsForUserCommand({ UserName: user.User.UserName }),
      ); // 사용자가 속한 모든 그룹 목록 검색

      let groupPolicies = [];
      for (const group of groups.Groups) {
        const attachedPolicies = await this.client.send(
          new ListAttachedGroupPoliciesCommand({ GroupName: group.GroupName }),
        ); // 그룹에 할당된 모든 IAM 정책 목록 검색
        groupPolicies.push(attachedPolicies);
      }

      return { user, policies, accessKeys, groups, groupPolicies };
    } catch (error) {
      console.error(`error = `, error);
    }
  }
```

> `GetCallerIdentityCommand`는 **현재 사용자의 ARN을 반환**하며, `AssumeRoleCommand`는 **현재 사용자의 임시 자격 증명을 반환**한다.
> 

> `ListGroupsForUserCommand`는 **사용자가 속한 모든 그룹 목록을 검색**하고, `ListAttachedGroupPoliciesCommand`는 **그룹에 할당된 모든 IAM 정책 목록을 검색**한다.
> 

> `groupPolicies` 배열에 그룹에 할당된 모든 IAM 정책을 추가하여 반환한다.
> 

```json
{
  "user": {
      "$metadata": {
          "httpStatusCode": 200,
          "requestId": "xxxx",
          "attempts": 1,
          "totalRetryDelay": 0
      },
      "User": {
          "Path": "/",
          "UserName": "xxxx",
          "UserId": "xxxx",
          "Arn": "arn:aws:iam::345619873383:user/xxxx",
          "CreateDate": "2023-03-20T00:48:54.000Z"
      }
  },
  "policies": {
      "$metadata": {
          "httpStatusCode": 200,
          "requestId": "xxxx",
          "attempts": 1,
          "totalRetryDelay": 0
      },
      "AttachedPolicies": [
          {
              "PolicyName": "AmazonEC2ReadOnlyAccess",
              "PolicyArn": "arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess"
          },
          {
              "PolicyName": "IAMFullAccess",
              "PolicyArn": "arn:aws:iam::aws:policy/IAMFullAccess"
          },
          {
              "PolicyName": "AmazonVPCReadOnlyAccess",
              "PolicyArn": "arn:aws:iam::aws:policy/AmazonVPCReadOnlyAccess"
          },
          {
              "PolicyName": "AmazonRDSReadOnlyAccess",
              "PolicyArn": "arn:aws:iam::aws:policy/AmazonRDSReadOnlyAccess"
          },
          {
              "PolicyName": "AmazonS3ReadOnlyAccess",
              "PolicyArn": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
          }
      ],
      "IsTruncated": false
  },
  "accessKeys": {
      "$metadata": {
          "httpStatusCode": 200,
          "requestId": "xxxx",
          "attempts": 1,
          "totalRetryDelay": 0
      },
      "AccessKeyMetadata": [
          {
              "UserName": "xxxx",
              "AccessKeyId": "xxxx",
              "Status": "Active",
              "CreateDate": "2023-03-20T07:07:38.000Z"
          }
      ],
      "IsTruncated": false
  },
  "groups": {
      "$metadata": {
          "httpStatusCode": 200,
          "requestId": "xxxx",
          "attempts": 1,
          "totalRetryDelay": 0
      },
      "Groups": [],
      "IsTruncated": false
  },
  "groupPolicies": []
}
```

## 클라우드 서버 IAM

```tsx
async getServerIAMInfo() {
  /**
   * 서버 접근통제 - IAM Policy & Role 정보
   */
  const policyCommand = new ListPoliciesCommand({});
  const policies = await this.client.send(policyCommand);
  // console.log(response.Policies);

  // IAM Policy에서 위에서 받아온 모든 정책들 중 서버에 해당하는 것들만 걸러내기
  const serverPolicies = policies.Policies.filter(
    (policy: any) =>
      policy.PolicyName.includes('EC2') ||
      policy.PolicyName.includes('RDS') ||
      policy.PolicyName.includes('Redshift'),
  );

  const roleCommand = new ListRolesCommand({});
  const roles = await this.client.send(roleCommand);

  // 마찬가지로 IAM Role에서 서버에 해당하는 것들만 걸러내기
  const serverRoles = roles.Roles.filter(
    (role: any) =>
      role.RoleName.includes('EC2') ||
      role.RoleName.includes('RDS') ||
      role.RoleName.includes('Redshift'),
  );

  return { serverPolicies, serverRoles };
}
```

> AWS에서 서버에 해당하는 IAM 정보를 필터링하기 위한 코드이다. 대표적으로 EC2, RDS, Redshift가 포함되는 경우 서버 관련 정책 및 역할 정보를 가져 올 수있다.
> 

```json
{
    "serverPolicies": [
       ...
        {
            "PolicyName": "AmazonRedshiftReadOnlyAccess",
            "PolicyId": "xxx",
            "Arn": "arn:aws:iam::aws:policy/AmazonRedshiftReadOnlyAccess",
            "Path": "/",
            "DefaultVersionId": "v1",
            "AttachmentCount": 0,
            "PermissionsBoundaryUsageCount": 0,
            "IsAttachable": true,
            "CreateDate": "2015-02-06T18:40:51.000Z",
            "UpdateDate": "2015-02-06T18:40:51.000Z"
        }
    ],
    "serverRoles": [
        {
            "Path": "/aws-service-role/rds.amazonaws.com/",
            "RoleName": "AWSServiceRoleForRDS",
            "RoleId": "xxx",
            "Arn": "arn:aws:iam::xxx:role/aws-service-role/rds.amazonaws.com/AWSServiceRoleForRDS",
            "CreateDate": "2018-04-17T01:45:51.000Z",
            "AssumeRolePolicyDocument": "%7B%22Version%22%3A%222012-10-17%22%2C%22Statement%22%3A%5B%7B%22Effect%22%3A%22Allow%22%2C%22Principal%22%3A%7B%22Service%22%3A%22rds.amazonaws.com%22%7D%2C%22Action%22%3A%22sts%3AAssumeRole%22%7D%5D%7D",
            "Description": "Allows Amazon RDS to manage AWS resources on your behalf",
            "MaxSessionDuration": 3600
        },
        ...
    ]
}
```
