# AWS - EC2 & Load Balancer 에서 생긴 문제점

## 에러 상황
서버와 웹 모두 ec2에서 설정이 완료된 후 izero.cloud로 접속해봤는데 hybrix.site로 설정했던 화면이 나왔다.

## 해결 방법

    izero.cloud와 hybrix.site는 같은 Load balancer를 사용하고 있는데 해당 로드 발란서에서 리스너 설정이 izero.cloud에 대해 없는 상태였기 때문에 그런 것으로 예상된다.

    그리고 리스너 설정을 하기 전 SSL 설정을 위한 인증서를 따로 생성해야 하기 때문에 AWS Certificate Manager에서 [izero.cloud](http://izero.cloud)에 대한 인증서를 발급하면 된다.
    
    해당 인증서는 cloudflare에서 Origin Server를 통해 발급할 수 있다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a1fbd910-852a-419d-8d1f-0397a38d1ce0/Untitled.png)