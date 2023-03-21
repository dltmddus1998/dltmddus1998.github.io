# Credential Report

> IAM 계정 관련하여 credential report를 생성하고 조회하는 기능을 구현할건데 이 때 node에서 사용하는 라이브러리가 `@aws-sdk/client-iam`이다
> 

```tsx
import {
  IAMClient,
  GenerateCredentialReportCommand,
  GetCredentialReportCommand,
} from '@aws-sdk/client-iam';

@Injectable()
export class AwsService {
  private readonly client: IAMClient;

  constructor() {
    this.client = new IAMClient({
      region: 'ap-northeast-2',
      credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      },
    });
  }
	.
	.
	.
}
```

```tsx
@Injectable()
export class AwsService {
	.
	.
	.

  async generateCredentialReport(): Promise<any> {
    const command = new GenerateCredentialReportCommand({});
    const response = await this.client.send(command);
    console.log(response.State);
    return response;
  }

  async getCredentialReport(): Promise<any> {
    const command = new GetCredentialReportCommand({});
    const response = await this.client.send(command);

    return csvJSON(new TextDecoder('utf-8').decode(response.Content));
  }
}
```

> `getCredentialReport`를 통해 조회한 정보는 csv 형식으로 반환되므로 이를 JSON 형식으로 변환하는 메서드를 생성하여 적용시켜 반환한 것이다.
> 

```tsx
export const csvJSON = (csv: string) => {
  var lines = csv.split('\n');

  var result = [];

  var headers = lines[0].split(',');

  for (var i = 1; i < lines.length; i++) {
    var obj: any = {};
    var currentline = lines[i].split(',');

    for (var j = 0; j < headers.length; j++) {
      obj[headers[j]] = currentline[j];
    }

    result.push(obj);
  }

  return result;
};
```

### Credential Report 정보 생성

```json
{
  "result": {
      "$metadata": {
          "httpStatusCode": 200,
          "requestId": "xxx",
          "attempts": 1,
          "totalRetryDelay": 0
      },
      "State": "COMPLETE"
  },
  "message": "🔑 Credential report generated successfully❕"
}
```

### Credential Report 정보 조회

```json
{
  "report": [
      {
          "user": "<root_account>",
          "arn": "arn:aws:iam::xxx",
          "user_creation_time": "2017-12-20T03:07:55+00:00",
          "password_enabled": "not_supported",
          "password_last_used": "2023-03-20T00:39:48+00:00",
          "password_last_changed": "not_supported",
          "password_next_rotation": "not_supported",
          "mfa_active": "true",
          "access_key_1_active": "false",
          "access_key_1_last_rotated": "N/A",
          "access_key_1_last_used_date": "N/A",
          "access_key_1_last_used_region": "N/A",
          "access_key_1_last_used_service": "N/A",
          "access_key_2_active": "false",
          "access_key_2_last_rotated": "N/A",
          "access_key_2_last_used_date": "N/A",
          "access_key_2_last_used_region": "N/A",
          "access_key_2_last_used_service": "N/A",
          "cert_1_active": "false",
          "cert_1_last_rotated": "N/A",
          "cert_2_active": "false",
          "cert_2_last_rotated": "N/A"
      },
			.
			.
			.
	]
}
```
