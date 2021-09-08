# Docker Container 내부에서 Host로 접근하는 방법(feat.macOS)

<!--more-->

## 개요

SSH Tunneling을 통해 특정 애플리케이션에 접근해야하는 경우가 있을 수 있다. 예를들어, VPC 내부에서만 접근할 수 있는 RDS가 있다고 한다면 호스트에서 바로 RDS로 접근은 불가능하다. RDS에 접근하는 방법으로는 VPC 내부에 있는 EC2(외부에서 접속이 가능한)를 통해서 접근하는 방법이 있다. 이를 SSH Tunneling으로 특정 포트를 포워딩 시켜둔다면 호스트에서는 `localhost:{특정 포트}`를 사용해 RDS를 접근할 수 있게 된다.

위의 예시 상황에서 RDS에 접근해야하는 곳이 로컬(macOS)가 아닌 로컬에서 실행중인 도커 컨테이너이라고 가정해보자. 만약 network의 세팅값을 특별히 지정하지 않았다면 도커 컨네이너에서 `localhost:{특정 포트}`를 사용해 RDS를 접근할 수 없을 것이다. 

필자는 이러한 상황에 부딛혀 도커 컨테이너에서 로컬(macOS) 호스트로 접근하는 방법에 대해 찾아보게 되었으며 그에 대한 내용을 정리해보고자 한다.



## 두 가지 방법에 대하여..

### 1. `host` 네트워크 드라이버 사용

docker는 여러가지의 network 방식을 제공하는데 bridge와 host에 대해 간단히 정리해보면 다음과 같다.

- bridge network
  <img width="970" alt="스크린샷 2021-06-23 오후 9 41 25" src="https://user-images.githubusercontent.com/37801041/123098124-c5336100-d46b-11eb-9b34-b5b183e7406b.png">

  - `docker0`이라는 bridge(`172.17.0.1`)가 생성되고 컨테이너들이 이 `docker0` 으로 연결이 되고 이를 통해 외부 또는 내부 컨테이너간의 접근이 가능하다.
  - Host 네트워크와 별개의 독립적인 네트워크를 구성할 수 있다.

- host network

  <img width="904" alt="스크린샷 2021-06-23 오후 8 55 54" src="https://user-images.githubusercontent.com/37801041/123092426-68cd4300-d465-11eb-8f3f-60d80ae77325.png">

  - 컨테이너들이 독립적인 네트워크 영역을 갖지 않고 host(macOS)의 네트워크를 같이 사용한다.
  - `--net=host` 옵션으로 지정할 수 있다.



위에서 두 가지 네트워크 방식을 살펴보았는데 '개요'에서 해결하고자 한 문제는 host 네트워크 방식을 사용한다면 `localhost:{특정 포트}` 를 통해 접속이 가능하게 된다. 하지만 도커 컨테이너 네트워크 영역을 가질 수 없다는 단점이 있다. 또!! 큰 단점이 하나가 더 있다. 바로 이 Host 네트워크 모드는 mac에서는 동작하지 않는다는 것이다. 결론적으로 macOS에서 컨테이너 내부에서 로컬(macOS) 호스트로 접근하기 위해서는 2번 방법을 사용해야한다.



### 2. `host.docker.internal` 도메인 사용

Docker는 자체적으로 `host.docker.internal`라는 도메인 네임을 가상으로 제공해준다. 이 도메인 네임을 이용하면 로컬(macOS) 호스트 영역으로 접근을 할 수 있다.

이를 사용한 예를 간단히 보이면 아래와 같다.

<img width="670" alt="스크린샷 2021-06-23 오후 9 19 46" src="https://user-images.githubusercontent.com/37801041/123095181-be571f00-d468-11eb-93a1-622cca536db2.png">

- 로컬 호스트에서 SSH Tunneling을 위해 20000번 포트로 포트포워딩을 해둔 상태라면 도커 컨테이너에서 `host.docker.internal:20000` 을 통해 RDS로 접속이 가능해진다.



## 결론

이번 글에서는 macOS의 도커 컨테이너 내부에서 로컬(macOS) 호스트로 접근하는 방법에 대하여 알아본 내용을 정리해보았다. 리눅스 머신을 사용한다면 host 네트워크 방식을 사용할 수 있겠지만 macOS에서는 이 host 네트워크 방식이 동작하지 않는다는 점이 있고 독립적인 네트워크를 사용할 수 없다는 단점을 가지고 있었다. 따라서 개요에서와 같은 문제에 부딛혔던 필자는 도커가 제공하는 `host.docker.internal`이라는 도메인 네임을 이용하므로써 의외로 간단하게 문제를 해결할 수 있었다. 이와 같은 경우가 빈번하지는 않을 것 같지만 해당 내용을 알고 있다면 필요한 상황에서는 꽤 유용하게 사용될 수 있을 것 같다.



## 참고

- https://seorenn.tistory.com/20
- https://bluese05.tistory.com/38
- https://docs.mirantis.com/containers/v3.1/dockeree-ref-arch/networking/scalable-container-networks.html




