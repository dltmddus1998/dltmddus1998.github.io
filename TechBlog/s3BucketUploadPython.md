# Purpose

> UiPath에서 크롤링 자동화를 이용하여 받아온 csv파일들을 AWS S3의 Bucket에 자동으로 주기적인 업로드를 구현하기 위한 방법을 찾던 중 해당 구현이 바탕이될 수 있을 것 같아 시도하게 됐다.
> 

# S3 Bucket 생성하기

💡 **AWS S3에서 내가 사용할 Bucket을 생성한 후 해당 Bucket을 활용하면 된다.**

```python
import boto3

bucket_name = '<내가 만들 버켓 이름>';

# 업로드하고자 하는 파일명
in_file = 'test1.csv';

# s3에 다 올려졌을 때의 파일명
out_file = in_file

s3 = boto3.client('s3')

s3.upload_file('./upload_data_to_s3.py', bucket_name, 'csv/' + out_file)
```

> 이후 위 파일을 실행해주면 해당 S3 bucket에 업로드돼있음을 확인할 수 있다.
> 

```python
import boto3
import botocore

bucket_name = '<다운로드 받고자 하는 bucket명>'

# s3에 존재하는 파일명
in_file = 'test1.csv'

out_file = in_file

s3 = boto3.resource('s3')

try:
    s3.Bucket(bucket_name).download_file('csv/' + in_file, out_file)
except botocore.exceptions.ClientError as e:
    if e.response['Error']['Code'] == '404' :
        print('해당 파일이 S3에 없습니다.')
    else:
        raise
```

> 이후 위 파일을 실행해주면 해당 S3 bucket에 있는 파일이 로컬에 다운로드됨을 확인할 수 있다.
>
