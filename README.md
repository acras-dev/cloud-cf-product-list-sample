# Cloud Foundry 응용 프로그램 샘플
샘플 응용 프로그램은 SAP Cloud Platform Cloud Foundry Environment에서 실행되고 일부 서비스를 사용합니다.

이 튜토리얼에서는 다음 작업을 수행하는 방법을 보여줍니다.
* SAP Cloud Platform Cloud Foundry Environment 평가판 계정에 접근하는 방법
* 백업 서비스와 같은 Cloud Foundry의 기본 개념을 사용하여 응용 프로그램을 처음부터 개발하고 추가 기능으로 향상시키는 방법
* SAP Cloud Platform 조종석 및 CF Foundstone CLI (Cloud Foundry Command Line Interface)를 사용하여 Cloud Foundry 평가판 계정 및 응용 프로그램과 연동하는 방법
* Spring과 Spring Boot와 같은 응용 프로그램 프레임 워크를 사용하여 응용 프로그램을 효율적으로 개발하는 방법
* 응용 프로그램을 모니터링, 확장 및 업데이트하는 방법


# 시나리오
당신의 상사가 어느날 아침에 와서 회사가 현대적인 방식으로 제품을 판매 할 수 있도록 새로운 전자 상거래 사이트를 개발해야한다고 말합니다. 당신의 회사는 이미 SAP Cloud Platform을 선택하여 사용하고 있지만 아직 경험이 많이 없습니다. 최근에 도입 된 Cloud Foundry Environment를 처음 접했을 것입니다. 클라우드 기반 응용 프로그램 개발을 위한 적절한 선택이라고 들었으므로 매우 빨리 시작하고 알아내야 합니다. 상사는 가능한 한 빨리 SAP Cloud Platform에서 실행되는 전자 상거래 사이트의 프로토 타입을 가져와 영업 부서에 보여주고 다음 단계를 논의 할 수 있기를 원합니다. 그의 초기 요구 사항은 다음과 같습니다.
* 제품 카탈로그에서 사용 가능한 제품 목록을 표시하는 기본 UI가 있어야합니다.
* 제품을 선택하면 그에 대한 전용보기가 제공됩니다.

따라서 SpringBoot로 프로토 타입 응용 프로그램을 개발할 것입니다.

당신은 PoC처럼 시작하여 운영 응용 프로그램으로 쉽게 진화되는 프로젝트 경험이 있기 때문에 응용 프로그램 모니터링 및 운영 옵션을 살펴보고 싶습니다. 영업 팀이 프로토 타입을보고 나면 실제 운영프로그램에 거의 가깝다고 생각해 가능한 한 빨리 사용하고 싶어한다는 것을 알고 있기 때문에, 출시 전에 효율적인 개발 작업을 위해 응용 프로그램을 준비하는 것이 무엇을 의미하는지 알고 싶습니다. 그래서 응용 프로그램을 관찰하고 확장하고 업데이트하는 옵션이 무엇인지보고 싶습니다.

다음 단계로 유연한 인증 프레임 워크 인 OAuth 2.0을 사용하여 Product List 응용 프로그램을 보호해야합니다. OAuth 2.0의 인증 코드 부여는 승인 된 사용자에게만 응용 프로그램 및 데이터에 대한 액세스 권한을 부여하는 탁월한 보안 메커니즘을 제공합니다. SAP XS Advanced Application Router, SAP XS UAA OAuth 인증 서비스 및 Spring Boot를 사용하면 손쉽게 역할을 구성하고 사용자에게 할당 한 다음 응용 프로그램에서 역할 검사를 구현할 수있는 탁월한 도구가 제공됩니다.

마지막으로 Cloud Connector에서 공개되는 사내 구축 형 시스템에 액세스하려고합니다. 연결 서비스로 응용 프로그램을 확장하여 해당 시스템에서 서비스를 사용할 수 있습니다.


# 구성 요소 개요
<br><br>
![Components Overview](/img/overview_components.png?raw=true)
<br><br>

위의 그림에 표시된 관련 소프트웨어 구성 요소는 개발 도구 및 각 세션의 구성 요소 세트 (기본 및 고급)입니다. 아래에서 자세한 내용을 읽을 수 있습니다.

연습 문제를 해결하려면 로컬 개발자에게이 구성 요소가 필요합니다. TechEd가 제공 한 랩탑을 사용한다면 이미 설치 및 구성되어 있어야합니다.

미니 코드 잼 및 기본 핸즈 온 :
- 이클립스
- CF CLI
- Maven, Git, Java

고급 실습 :
- MTA 아카이브 빌더
- SAP 클라우드 커넥터


## 기본 실습 세션

**제품 목록 신청**

연습 도중 개발되고 향상 될 샘플 응용 프로그램입니다. SAP Cloud Platform Cloud Foundry에서 실행됩니다. 클라우드의 지속성 계층 용 PostgreSQL, 응용 프로그램 로깅 등 간단한 UI로 SpringBoot 응용 프로그램입니다.

**PostgreSQL**

PostgreSQL은 오픈 소스 객체 관계형 데이터베이스 관리 시스템입니다. 엔터프라이즈 급 ACID 호환 데이터베이스입니다. Cloud Foundry Environment에서 백업 서비스로 사용할 수 있습니다.

**응용 프로그램 로깅 서비스**

응용 프로그램 로깅 서비스를 사용하여 응용 프로그램 로그를 작성, 액세스 및 분석 할 수 있습니다. 오픈 소스 로깅 플랫폼 Elasticsearch, Logstash, Kibana (Elastic Stack)를 기반으로합니다. Cloud Foundry 구성 요소에서 비롯된 응용 프로그램 로그와 응용 프로그램에서 명시 적으로 발급 한 로그를 모두 가질 수 있습니다. 응용 프로그램 로그를 응용 프로그램 로깅 서비스로 스트리밍하려면 응용 프로그램을 준비해야합니다.

**응용 프로그램 자동 스케일러 서비스**

응용 프로그램 자동 스케일러 서비스는 사용자 정의 정책을 기반으로 바운드 응용 프로그램 인스턴스를 자동으로 확장 또는 축소하는 데 사용됩니다. 응용 프로그램 인스턴스를 동적으로 확장하면 응용 프로그램이 충돌하지 않거나로드가 증가 할 때 성능 문제가 발생하지 않습니다. 로드가 줄어들면 동적으로 축소되므로 응용 프로그램이 최적의 리소스를 사용합니다.

## 고급 실습 세션

위에서 설명한 구성 요소 외에도 고급 연습에서 더 많은 서비스를 탐색하고 사용 및 구성하여 제품 목록 샘플 응용 프로그램에보다 정교한 기능을 추가하는 방법을 살펴 봅니다.

**XS UAA 서비스**

XS UAA (User Account and Authentication) 서비스는 멀티 파운드 (multi-tenant) ID 관리 서비스로 Cloud Foundry에서 사용됩니다. 주요 역할은 OAuth 2.0 공급자로서 클라우드 Foundry 사용자를 대신하여 클라이언트 응용 프로그램이 작동 할 때 사용할 토큰을 발급하는 것입니다. 또한 Cloud Foundry 자격 증명으로 사용자를 인증 할 수 있으며 해당 자격 증명 (또는 다른 자격 증명)을 사용하여 SSO 서비스로 작동 할 수 있습니다. 또한 사용자 계정을 관리하고 OAuth 2.0 클라이언트를 등록하기위한 엔드 포인트를 비롯하여 다양한 관리 기능을 제공합니다.


**응용 프로그램 라우터**

마이크로 서비스 아키텍처를 포용하는 비즈니스 응용 프로그램은 독립적으로 관리 할 수있는 여러 서비스로 분해됩니다. 이 접근 방식은 일관된 방식으로 보안을 처리하고 동일한 출처 정책을 다루는 것과 같은 응용 프로그램 개발자에게 몇 가지 과제를 제기합니다. Application Router는 브라우저에 의해 액세스되는 엔드 포인트를 노출하여 응용 프로그램에 액세스하고 이러한 문제점 중 일부를 해결하는 별도의 구성 요소입니다. 세 가지 주요 기능을 제공합니다.
- 역방향 프록시 - 비즈니스 응용 프로그램에 대한 단일 진입 점을 제공하고 사용자 요청을 각각의 백엔드 서비스에 전달합니다.
- 파일 시스템에서 정적 컨텐츠를 제공합니다.
- 보안 - 중앙 위치에서 로그인, 로그 아웃, 사용자 인증, 권한 부여 및 CSRF 보호와 같은 보안 관련 기능을 제공합니다.


**연결 서비스**

연결 서비스는 모든 응용 프로그램에서 액세스 할 수있는 구내 연결을위한 표준 HTTP 프록시를 제공합니다. 프록시 호스트 및 포트는 변수로 사용할 수 있습니다. 응용 프로그램은 SAP-Connectivity-Authentication 헤더를 통해 사용자 JWT 토큰을 전파해야합니다. 이는 클라우드 커넥터에서 구성이 이루어진 하위 계정에 대한 터널을 열려면 연결 서비스에서 필요합니다.


# 세션 수

미니 코드 잼 (1 시간), 기본 실습 (2 시간), 고급 실습 (2 시간)의 3 가지 세션이이 샘플 응용 프로그램을 사용합니다. 아래의 해당 링크를 따라 참석 한 세션에 대한 자세한 연습을 찾을 수 있습니다.

:one:**미니 코드잼 (1 시간)**

이 세션에서는 조종실 및 Cloud Foundry CLI에 익숙해 질 것입니다. 특히, 다음을 배우게됩니다.
- 응용 프로그램을 올리십시오.
- 서비스를 앱에 바인딩
- 앱 로그 확인
- 상태 확인 변경
- 푸른 녹색 배치

이 세션 세부 단계 설명 : [Mini CodeJam](/exercises/basic-codeJam)

:two:**기본 실습 (2 시간)**

이 세션에서는 간단한 SpringBoot 응용 프로그램을 개발하고 로컬로 실행 한 다음 SAP Cloud Platform Cloud Foundry Environment로 푸시합니다. 응용 프로그램에 대한 내용을 탐색하고 기본 로그 및 메트릭을 확인한 후에는 지원 기능과 함께 응용 프로그램을 향상시킬 수 있습니다. 응용 프로그램 로깅 서비스와의 통합 또한 응용 프로그램을 확장하는 방법을 볼 수 있습니다.

이 세션의 세부 단계 설명 : [Basic Hands-on](exercises/basic-hands-on)

:three:**고급 실습 (2 시간)**

이 세션에서는 샘플 Product List 응용 프로그램의 좀 더 정교한 향상 및 작동을 살펴 봅니다.
* 청색 / 녹색 배치 방식을 사용하여 Cloud-Foundry에서 실행되는 응용 프로그램을 업데이트하는 방법을 알아보십시오.
* MTA 아카이브를 만들고 여러 모듈로 구성되어 있고 여러 서비스를 사용하는보다 복잡한 응용 프로그램이있는 경우 배포 및 업데이트가 어떻게 단순화되는지 확인하십시오.
* 제품 목록 응용 프로그램을 보안하고 OAuth 2.0 인증 코드 부여 (사람과 서비스 간 통신)를 구성합니다.
* 클라우드 커넥터에 의해 노출 된 온 - 프레미스 시스템에 연결성 서비스를 액세스하고 액세스하도록 응용 프로그램을 확장합니다.

이 세션의 세부 단계 설명 : [Advanced Hands-on](exercises/advanced-hands-on)
