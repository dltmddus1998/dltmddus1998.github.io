# Amazon의 Glacier에 대하여 (무한대로 저장 가능한 스토리지 만들기)

## Amazon Glacier

> **Amazon Glacier**는 데이터 아카이빙 및 장기 백업을 위한 안전하고 안정적이며 비용이 매우 저렴한 클라우드 스토리지 서비스이다.
> 

> 99.99%의 안정성을 제공하도록 설계되어 있으며, 가장 엄격한 데이터 보관에 대한 규제 요구사항도 충족할 수 있는 종합적인 보안 및 규정 준수 기능을 제공한다.
> 

✔️ S3의 개별 스토리지 영역인 ‘Bucket’과 유사한 ‘**Vault**’라는 개별 스토리지 영역을 생성하여 데이터를 보관하며, Console을 통한 업로드로 지원하며 별도의 API를 이용하여 데이터에 대한 저장 기능을 제공한다. 

일반적으로 S3에 저장되는 데이터는 라이프사이클 옵션을 활용하여 일정 기간 이상 지난 데이터에 대해 보다 저렴한 Glacier로 이동하여 저장하는 옵션을 사용할 수 있다. 

## Amazon Glacier의 주요 특징

### Amazon Glacier의 데이터 접근 방법

- API/SDK를 이용한 Direct 연결
    - API나 SDK를 활용한 프로그램 개발을 통해 깊게 저장된 데이터를 위한 Glacier에 직접 접속
- S3 라이프 사이클과의 통합
    - S3의 라이프 사이클과 통합을 통해 오래된 데이터에 대해 Glacier로 자동 이관
- 3rd Party Tool과 AWS Storage Gateway 연동
    - 기존 Backup 인프라와 3rd Party Tool과의 연계 및 AWS Storage Gateway 통합을 통해 거부감 없는 방식으로 데이터 백업 및 보관 기능 제공
