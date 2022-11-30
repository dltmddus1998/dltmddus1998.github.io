# aws-sdk를 이용하여 rds 데이터 다루기

## @aws-sdk/client-rds 사용 예시

```tsx
// ⭐️ @aws-sdk/client-rds Example
  const client = new RDSClient({ region: 'ap-northeast-2' });

  const params = {
    /** input parameters */
    DBClusterIdentifier: 'practice1',
    RoleArn: 'arn:aws:iam::345619873383:role/rds-monitoring-role',
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
export interface AddRoleToDBClusterMessage {
  /**
   * <p>The name of the DB cluster to associate the IAM role with.</p>
   */
  DBClusterIdentifier: string | undefined;
  /**
   * <p>The Amazon Resource Name (ARN) of the IAM role to associate with the Aurora DB
   *             cluster, for example <code>arn:aws:iam::123456789012:role/AuroraAccessRole</code>.</p>
   */
  RoleArn: string | undefined;
  /**
   * <p>The name of the feature for the DB cluster that the IAM role is to be associated with.
   *             For information about supported feature names, see <a>DBEngineVersion</a>.</p>
   */
  FeatureName?: string;
}
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

```tsx
resourceArn: string | undefined;
/**
 * <p>The ARN of the secret that enables access to the DB cluster. Enter the database user name and password for the credentials in
 *             the secret.</p>
 *         <p>For information about creating the secret, see <a href="https://docs.aws.amazon.com/secretsmanager/latest/userguide/create_database_secret.html">Create a database secret</a>.</p>
 */
secretArn: string | undefined;
/**
 * <p>The SQL statement to run. Don't include a semicolon (;) at the end of the SQL statement.</p>
 */
sql: string | undefined;
/**
 * <p>The name of the database.</p>
 */
database?: string;
/**
 * <p>The name of the database schema.</p>
 *         <note>
 *             <p>Currently, the <code>schema</code> parameter isn't supported.</p>
 *         </note>
 */
schema?: string;
/**
 * <p>The parameter set for the batch operation.</p>
 *         <p>The SQL statement is executed as many times as the number of parameter sets provided.
 *           To execute a SQL statement with no parameters, use one of the following options:</p>
 *         <ul>
 *             <li>
 *                 <p>Specify one or more empty parameter sets.</p>
 *             </li>
 *             <li>
 *                 <p>Use the <code>ExecuteStatement</code> operation instead of the <code>BatchExecuteStatement</code> operation.</p>
 *             </li>
 *          </ul>
 *         <note>
 *             <p>Array parameters are not supported.</p>
 *         </note>
 */
parameterSets?: SqlParameter[][];
/**
 * <p>The identifier of a transaction that was started by using the
 *                 <code>BeginTransaction</code> operation. Specify the transaction ID of the
 *             transaction that you want to include the SQL statement in.</p>
 *         <p>If the SQL statement is not part of a transaction, don't set this
 *             parameter.</p>
 */
transactionId?: string;
```
