# UiPath로 특정 검색어에 해당하는 기사들을 크롤링하여 csv 파일로 추출한 후 AWS S3 Bucket으로 주기적으로 업로드하기 (ing)

<aside>
💡 위 방식의 심화 과정으로, 일정 시간마다(ex. 1시간마다) 해당 데이터를 다른 파일명으로 (ex. 삼성1.csv, 삼성2.csv …) 추출해낸 후, 이를 AWS S3 Bucket에 업로드해보라.

</aside>

## 구현 방법

1. uipath를 이용하여 csv로 특정 기사 크롤링 한 것을 추출 하는 과정을 자동화한 후, uipath의 cron 기능을 이용해 이를 주기적으로 가능하게 해준다.
    1. 이 때 csv 파일들이 중복되는 것이 아닌, “samsung1.csv, samsung2.csv…”와 같이 다른 이름의 파일명들이 나오도록 해야한다.
2. 이 후 해당 csv 파일들을 주기적으로 AWS S3 bucket에 업로드할 수 있는 python 코드를 짠다.

## 구현 과정

## 실행 결과

## 참고 자료

[Python을 이용하여 로컬에 있는 파일을 S3 Bucket에 업로드하기](https://www.notion.so/Python-S3-Bucket-d8788f6509474b14a909cc6d449248ca) 

[Using Cron Expressions](https://docs.uipath.com/orchestrator/docs/using-cron-expressions)

[https://www.youtube.com/watch?v=w1NrbhwtHCQ](https://www.youtube.com/watch?v=w1NrbhwtHCQ)
