# Nest Microservices - gRPC, API Gateway (ing)
## Microservice

<aside>
💡 애플리케이션이 서비스들의 컬렉션으로 개발되는 어플리케이션 아키텍쳐를 의미한다.

</aside>

> microservice들의 아키텍쳐 다이어그램들과 서비스들을 독립적으로 개발하고 배포하고 유지하기 위해 microservice architecture은 프레임워크를 제공한다.
> 

## Nest Microservice

<aside>
💡 Nest에서 microservice는 다른 transport 계층을 사용하는 HTTP보다 완전한 어플리케이션이다.

</aside>

## gRPC

### 1️⃣ gRPC 정의

<aside>
📌 gRPC는 현대적이고, 오픈 소스가 있으며, 높은 성능을 가진 RPC 프레임워크이며 어느 환경에서든 실행될 수 있다.

</aside>

### 2️⃣ gRPC 특징

✔️ 로드밸런싱, 트레이싱, 헬스 체킹, 그리고 인증에 대한 접속 가능한 지원과 함께 데이터 센터에서 서비스들을 효율적으로 연결할 수 있다. 

✔️ 많은 RPC와 같이, gRPC는 자동적으로 불릴 수 있는 메서드들의 용어에서 서비스를 정의하는 개념에 기반한다.

✔️ 서비스, 파라미터, 그리고 리턴 타입들은 구글의 오픈 소스 language-neutral protocol buffers mechanism을 이용하는 `.proto` 파일들에 정의되어 있다.

### 3️⃣ API Gateway

> API Gateway는 HTTP 기반의 클라이언트 요청들의 entry point이다. 그러나 HTTP만을 제한할 필요는 없다.
> 

API - 

✔️ API Gateway는 두가지 방법 중 하나를 통해 요청을 처리한다.

✔️ 몇몇 요청들은 단순히 하나의 서비스에 proxy / route된다.

✔️ 다른 요청들을 다수의 서비스들에 퍼뜨림으로써 처리한다.
