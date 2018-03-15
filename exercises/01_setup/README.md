# 환경 설정


## 예상 시간

:clock4: 10 분

## 목표

이 연습에서는 SAP Cloud Platform에서 무료 평가판 계정을 만들고, Cloud Foundry Environment 평가판을 시작하고, 로컬 개발 환경을 설정하고, SAP Cloud Platform 용 웹 기반 관리 도구 인 SAP Cloud Platform 조종석을 살짝 탐험하는 방법을 배우게 됩니다.

# 연습과정 설명

## 당신의 Cloud Foundry를 시작하세요.

:bulb: **Note:** 아직 SAP Cloud Platform에 대한 평가판 계정이없는 경우 이 [문서](https://www.sap.com/developer/tutorials/hcp-create-trial-account.html)를 수행하여 얻으십시오.

1. 그런 다음 [SAP Cloud Platform cockpit](https://account.hana.ondemand.com/#/home/welcome) 조종석에서 사용자 홈으로 이동하십시오. *Start Cloud Foundry Trial* 버튼이 표시 됩니다.
<br><br>
![Start Cloud Foundry Trial](/img/start_cf_trial.png?raw=true)
<br><br>
2. *Start Cloud Foundry Trial* 버튼을 클릭합니다. - 팝업이 나타납니다. 평가판의 드롭 다운에서 지역을 선택하고 확인을 클릭하십시오.
<br><br>
![Select Trial Region](/img/select_trial_region.png?raw=true)
<br><br>

3. 절차가 끝날 때까지 기다리십시오. 선택한 지역에서 글로벌 계정, 서브 계정, 조직 및 SPACE를 받게됩니다. 생성이 완료되면 새로운 SPACE로 이동할 수 있습니다 (Go to Space 버튼 클릭)
<br><br>
![Select Trial Region](/img/go_to_space.png?raw=true)
<br><br>

:bulb: **Note:** Cloud Foundry 환경 평가판을 이미 시작한 경우 Cloud Foundry Trial 시작 버튼 이 표시되지 않습니다 . 이 경우 Go to Cloud Foundry Trial 버튼을 클릭하면 됩니다.
<br><br>
![GO to trial](/img/go_to_trial_button.png?raw=true)
<br><br>
이제 당신의 Cloud Foundry 평가판 SPACE가 생겼습니다.

## 조종석
이것은 Cloud Foundry Environment에 있는 도메인 모델의 단순화 된 그림입니다. 다른 엔티티에 대해 더 자세히 알고 싶으면 [문서](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/8ed4a705efa0431b910056c0acdbf377.html)를 확인하십시오.

<br><br>
![Domain Model Overview](/img/domain_model.png?raw=true)
<br><br>

이것이 조종실에서 지금 볼 수있는 것입니다.
<br><br>
![Cockpit Domain Model Overview](/img/cockpit_domain_model.png?raw=true)
<br><br>

## 계정 정리

:bulb: **Note:** Cloud Foundry Environment 평가판 계정에는 제한된 용량이 있습니다.

이미 시험 사용 계정 할당량을 사용한 경우 연습을 계속하기 전에 응용 프로그램 및 서비스 인스턴스를 정리해야합니다. 조종실을 통해 그렇게 할 수 있습니다.

- Space에서 실행중인 응용 프로그램 목록으로 이동하여 실행중인 응용 프로그램을 모두 삭제하거나 실행중인 응용 프로그램을 모두 중지하십시오.
- 서비스 인스턴스로 이동하여 모두 삭제하십시오.

## 로컬 개발 환경

:bulb: **Note:** 로컬 개발 환경이 이미 설치되어 있는 경우 설치할 필요가 없습니다.

클라우드 파운드리 명령 줄 인터페이스 (CF CLI)
Cloud Foundry 명령 줄 인터페이스 (CF CLI) 설치 또는 업데이트

### 설치
CF CLI를 다운로드하고 설치 지침을 따르십시오 : [CF CLI](https://github.com/cloudfoundry/cli#downloads)

- Windows : Windows 64 비트 설치 프로그램을 다운로드하십시오.
- Mac : OS X 설치 프로그램을 다운로드하십시오. 참고 : 대안으로 Homebrew 오픈 소스 패키지 관리 소프트웨어를 사용하여 CLI를 다운로드 할 수 있습니다.
- Linux : Debian / Ubuntu 또는 Red Hat 시스템 용 Linux 설치 프로그램을 다운로드하십시오.

#### 업데이트
[CF CLI](https://github.com/cloudfoundry/cli#downloads) 바이너리를 다운로드하고 기존파일과 교체하십시오. Windows인 경우 `cf.exe`를 설치되어있는 폴더로 옮깁니다.

### 이클립스
STS (Spring Tool Suite) 플러그인을 설치해야합니다.

### Java
사용 가능한 Java 버전을 확인하십시오 `java -version`.

### Maven
- 실행으로 Maven이 올바르게 구성되어 있는지 확인하십시오. `mvn --version`
- 아직 설치되지 않았다면 설치 지침을 따릅니다. https://maven.apache.org/install.html

### Git
- 실행으로 Git이 올바르게 구성되어 있는지 확인하십시오. `git --version`
- 아직 설치되지 않았다면 설치 지침을 따릅니다. https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
