---
title:        "MicroService란 무엇인가?"
categories:
              - "MSA"
tags:         
              - "Spring"
              - "MSA"
              - "Architecture"
              - "MicroService"
---
### MicroService Architecture 등장 배경
* 비즈니스적 요구
  * `항공사의 예약 시스템`
  * `은행의 뱅킹 시스템`
  * `기존의 ERP...`
* 기술의 발전
  * `Cloud`
  * `Javascript Framework(Angular, Ember...)`
  * `NoSQL`

항공사의 예약시스템, 은행의 뱅킹시스템은 과거에 대표적인 일체형 애플리케이션으로 구현되어왔습니다. 그러나 시시각각 변하는 새로운 비즈니스 모델을 반영하기 위해서 이들은 MicroService Architecture를 도입하고 있습니다. Cloud, JS Framework, NoSQL과 같은 기술의 발전은 이러한 변화의 속도를 더욱 높이고 있습니다.

### MicroService란 무엇인가?

> MSA(MicroService Architecture) Style은 각자 별도의 프로세스에서 실행되며, HTTP 자원 API 같은 가벼운 메커니즘으로 통신하는 작은 서비스를 모아 하나의 애플리케이션을 만든다. 이런 작은 서비스들은 각자의 비즈니스 기능을 담당하고 완전 자동화된 절차에 따라 독립적으로 배포 가능하다. 작은 서비스를 관리하는데 중앙 집중형 관리 방식은 최소한으로 사용되며, 각 서비스는 서로 다른 프로그래밍 언어나 서로 다른 데이터 저장 기술을 사용할 수도 있다.

> Martin Fowler, http://www.martinfowler.com/articles/microservices.html

MicroService라는 것은 `Agile`, `신속성`, `확장성`을 확보할 수 있는 중요한 수단으로 사용 중인 아키텍처 스타일이라고 할 수 있습니다. `MicroService`는 물리적으로 분리할 수 있는 모듈화된 애플리케이션을 만들 수 있는 방법을 제시하고 있습니다.

### 기존 N-tear vs MSA 비교

![N-tear](/assets/images/N-tear_layers.jpg){: width="450px" height="300px"}
* 기존 N-tear Architecture는 각 모듈이 하나의 Layer로 통합된 모습입니다.

![MSA](/assets/images/MSA_layers.jpg){: width="450px" height="300px"}
* 반면 MSA의 경우에는 각각의 모듈별로 Layer가 구성됩니다.

### MicroService Architecture의 원칙
이러한 MSA를 구현하기 위해서는 지켜야할 두 가지 주요한 원칙이 있습니다.
1. 하나의 단위 요소는 하나의 책임만 가져야 한다
  * 하나의 단위 요소가 여러 개의 책임을 가지면 결국 다른 요소들과 지나치게 높은 결합도를 형성하기 때문입니다
2. 라이브러리 의존성을 포함한 모든 의존 관계와 웹서버나 컨테이너 또는 물리적인 자원을 추상화하는 가상머신 모두를 함께 갖고 있다.
  * 대부분의 서비스 지향 아키텍처 구현체가 서비스 수준의 추상화를 제공하는데 반해 MicroService는 한발 더 나아가 실행 환경까지도 추상화합니다

### MSA의 특징
* MSA는 가볍다
* 다양한 언어로 구성할 수 있다
* 개발부터 운영까지 전 과정을 최대한 자동화 한다
* 해당 비즈니스 영역의 데이터와 로직만을 포함한다
* `antifragility`, fail fast, self-healing
  * Simian Army(Netflix)
  * https://github.com/Netflix/SimianArmy
-> Chaos Monkey는 랜덤하게 가상 머신을 terminate 시켜서 신뢰성 보장 Test

### MSA의 장점
1. 폴리글랏(polyglot) Architecture 지원합니다.
  * 각각의 모듈별로 구현하기 때문에 프로그래밍 언어에 대한 제약을 받지 않습니다
  * 예를 들면 상품의 주문, 회원 관리 모듈은 `RDBMS`를 이용하고 감사를 위한 데이터 처리 모듈은 `hadoop`으로 구현할 수 있습니다
2. 실험과 혁신 유도
  * 새로운 기술로 구현된 다양한 기능을 마음대로 적용해 볼 수 있습니다
3. 탄력적이고 선택적인 확장
  * 전체 시스템에서 트랜잭션이 많은 개별 모듈만 Scale up, out 할 수 있습니다
4. 대체 가능성
  * 틈새 기능을 외부 솔루션이나 오픈소스로 사용하는 경우에 일체형 애플리케이션에 비해 서드파티 솔루션을 끼워넣기가 쉽다는 장점이 있습니다
5. `유기적 시스템 구축 유도`
  * 유기적 시스템이란 시간이 지남에 따라 점점 많은 기능이 더해서 성장하는 시스템을 말합니다. 시스템이 계속 커지면서 유지 관리성이 급격히 떨어지게 되는데 이를 MSA를 이용해서 효율적으로 관리할 수 있습니다.
6. 다양한 버전의 공존
  * 모듈별로 의존성을 관리하기 때문에 각각 다른 버전을 이용하더라도 서로간 영향을 미치지 않습니다
7. 이벤트 주도 아키텍쳐 지원

### 결론
MSA는 현재 IT시장에서 아주 Hot한 친구입니다. 모든 애플리케이션에 적용할 수 있는 아키텍처 모델은 아니지만 다양한 비즈니스 요구에 대응할 수 있는 최적의 방법 중 하나라고 생각합니다.

### Reference
이미지 출처링크 : https://www.packtpub.com/sites/default/files/downloads/Spring5.0MicroservicesSecondEdition_ColorImages.pdf

내용 출처 : 스프링5.0 마이크로서비스 2/E(에이콘 출판사)
