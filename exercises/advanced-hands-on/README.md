# 고급 실습 세션 - 2 시간

이 튜토리얼에서는 다음 작업을 수행하는 방법을 보여줍니다.
* SAP Cloud Platform Cloud Foundry 환경 시험판에 접근
* SAP Cloud Platform 조종석 및 Cloud Foundry CLI를 사용하여 평가판 작업 수행
* 응용 프로그램 보안 및 다양한 역할 구성
* 백엔드 시스템에서 데이터를 얻기 위해 연결을 설정하고 사용
* 응용 프로그램을 확장 및 업데이트

# 연습과정

:one: **[환경설정](../01_setup)**

이 연습에서는 응용 프로그램을 배포하고 실행하는 데 사용할 수있는 무료 SAP Cloud Platform Cloud Foundry Environment 평가판을 시작합니다. 또한 개발 환경과 도구를 설정합니다.

:two: **[샘플 응용 프로그램 복제(advanced branch)](../11_clonebranch)**

**advanced** branch 지점의 product list 응용 프로그램 복제

:three: **[OAuth 2.0으로 보안된 애플리케이션 만들기](../09_secure)**

Product List 응용 프로그램에 보안을 설정하고 OAuth 2.0 클라이언트 자격 인증 (서비스 통신에 대한 서비스) 및 OAuth 2.0 인증 코드 부여 (휴먼에서 서비스로의 통신)

:four: **[연결](../10_connectivity)**

클라우드 커넥터 및 연결 서비스를 사용해 온-프레미스 백엔드 시스템으로부터 데이터를 받습니다.

:five: **[(Optional) 스케일 조정](../07_scale)**

SAP Cloud Platform Cloud Foundry Environment에서 제공되는 다양한 응용 프로그램 확장 옵션을 살펴보십시오.

:six: **[(Optional) 업데이트](../08_update)**

응용 프로그램의 업데이트를 위해 CF-CLI 명령을 사용하여 수동으로 Blue-green deployment를 적용한 다음 MTA를 작성하고 MTA 플러그인 자동 Blue-green deployment를 사용합니다.
