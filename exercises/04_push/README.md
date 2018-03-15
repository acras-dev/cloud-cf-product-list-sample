# 연습과정 03: 응용 프로그램을 SAP Cloud Platform Cloud Foundry Environment로 올리기

## 예상 시간

:clock4: 10 분

## 목표
이 연습에서는 Cloud Foundry Environment 평가판 계정에서 SAP Cloud Platform의 응용 프로그램을 올릴 수있는 방법을 학습합니다. 우리는 Cloud Foundry Command Line Interface를 사용할 것입니다. 명령 프롬프트를 열고 아래 단계를 수행하십시오.

# 연습과정 설명
## 대상 & 로그인

:bulb: **Note** (Windows) 터미널을 실행하십시오. For that please:
	- 윈도우키 + 'R' 키 누르기
	- 입력창에 ```cmd``` 입력후 엔터

1. CF CLI에 사용할 Cloud Foundry를 알려줘야 합니다. 이렇게 하려면 Cloud Foundry 시험판을 만든 Cloud Foundry 지역의 Cloud Controller에 API endpoint를 설정해야 합니다.
```
cf api CLOUD_FOUNDRY_API_ENDPOINT
```

- US10 지역의 API endpoint 지정하는 방법:
```
cf api https://api.cf.us10.hana.ondemand.com
```
 - EU10 지역의 API endpoint 지정하는 방법:
```
cf api https://api.cf.eu10.hana.ondemand.com
```
:bulb: **Note:** [SAP Cloud Platform Documentation](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/350356d1dc314d3199dca15bd2ab9b0e.html)문서에서 Cloud Foundry Environment를 사용할 수 있는 여러 지역에 대한 API 엔드 포인트를 찾을 수 있습니다.

2. 사용자 계정으로 로그인하십시오. 명령 프롬프트에서 다음을 입력하십시오.
	```
	cf login
	```

	SAP Cloud Platform 평가판 계정에 등록 할 때 사용한 전자 메일 및 암호를 입력하라는 메시지가 나타납니다.

	```
	Email> enter your e-mail
	Password> password for your user
	```
3. 클라우드 파운드리 조직과 사용할 공간을 선택해야합니다. 하나의 Cloud Foundry 조직 및 공간에만 할당 된 경우 시스템은 로그인하면 자동으로 관련 Cloud Foundry 조직 및 공간을 대상으로 지정하며 이전 단계에서 OK 아래에 표시됩니다.

:bulb: **Note:** SAP Cloud Platform Cloud Foundry Environment의 이전 사용으로 인해 지정한 지역에 둘 이상의 Cloud Foundry 조직과 공간을 생성 한 경우 cf target 명령에서 어느 것을 사용해야하는지 선택하거나 프롬프트가 표시 될 때 선택합니다.
```
cf target -o ORGANIZATION -s SPACE
```

이제 Cloud Foundry 공간에서 작업 할 준비가되었습니다.

## 응용 프로그램 Manifest

응용 프로그램 Manifest 파일에 대한 자세한 내용은 [Cloud Foundry documentation](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)를 참조하십시오

* 이제 sample Product List 응용 프로그램에 대한 manifest.yml을 만듭니다. 이클립스로 다시 이동하고 Eclipse의 프로젝트 탐색기에서 sample Product List app -> 오른쪽 클릭 -> New -> File. 을 클릭한다. 파일을 만들고 이름을 **manifest.yml**로 하십시오.

* 이제 파일을 열고 응용 프로그램에 맞게 다음 스니펫을 삽입하십시오.

  :bulb: **Note** ```path``` Maven 빌드의 결과로 프로젝트의 대상 폴더에서 생성된 jar 파일을 가리켜야 합니다.

  :bulb: **Note** ```host``` 호스트 네임 스페이스가 Cloud Foundry 내의 모든 응용 프로그램과 공유되므로 값이 고유해야합니다. 따라서 생년월일같은 값을 뒷부분에 추가해 고유한값이 되도록 수정하십시오.

```Configuration
 applications:
 # Application
 - name: product-list
   instances: 1
   memory: 896M
   host: product-list-BIRTHDATE
   path: target/my-product-list-0.0.1-SNAPSHOT.jar
   buildpack: https://github.com/cloudfoundry/java-buildpack.git#v4.3
```
수정이 끝나면 저장합니다.

## 올리기
- 명령 프롬프트에서 manifest.yml 파일을 방금 작성한 SpringBoot 응용 프로그램의 루트 디렉토리로 이동하십시오. 다음 명령을 입력하십시오.
```
cf push
```
방금 만든 응용 프로그램 manifest.yml을 사용하여 응용 프로그램 배포 프로세스가 시작됩니다. 첫 번째 단계는 응용 프로그램 바이너리를 클라우드에 업로드하는 것입니다. 다음 단계에서는 소위 스테이징이 발생하여 :droplet:물방울이 생깁니다. 그런 다음 출시 단계에서는 Cloud Foundry에 응용 프로그램 시작 방법을 알려줍니다. 일단 명령이 성공하면 브라우저에서 응용 프로그램을 요청할 수 있습니다 - 콘솔에 인쇄 된 URL을 복사하여 붙여 넣기하십시오.

:bulb:**Note:** *command fails*이 발생한 경우, 오류 메시지를 읽으십시오. 로컬 샘플의 manifest.yml에서 매개 변수를 올바르게 변경하지 않았거나 평가판 계정의 할당량이 초과된 것일 수 있습니다. 이미 평가판을 사용했다면 연습을 계속하기 전에 응용 프로그램 및 서비스 인스턴스를 정리해야 합니다. 조종실을 통해 그렇게 할 수 있습니다.

## 응용 프로그램 호출하기
평가판 계정으로 SAP Cloud Platform Cloud Foundry Environment에서 실행되는 응용 프로그램을 사용할 수 있습니다. Chrome 브라우저를 열고 SAP Cloud Platform cockpit으로 이동합니다.
- https://account.hana.ondemand.com/cockpit#/home/overview
- Home 클릭
- Go to Cloud Foundry Trial 버튼 클릭
<br><br>
![GO to trial](/img/go_to_trial_button.png?raw=true)
<br><br>
- Global Account -> subaccount(일반적으로 trial) -> 왼쪽 메뉴에서 Spaces 클릭 -> 만들어져 있는 Space 클릭(일반적으로 dev)
<br><br>
![GO to space](/img/cockpit_CF_space_org.png?raw=true)
<br><br>
- 실행중인 응용 프로그램을 볼 수있는 평가판 space를 보고 있습니다. 방금 올린 product-list 응용 프로그램이 보여야 합니다. 응용 프로그램 이름을 클릭 할 수 있습니다. 전용 응용 프로그램보기가 열리며 응용 프로그램 경로가 표시됩니다. URL을 클릭하면 브라우저에서 실행중인 응용 프로그램을 볼 수 있습니다.
<br><br>
![Running applications in cockpit](/img/running_app_cockpit.png?raw=true)
<br><br>
전용 애플리케이션 보기가 열리며 애플리케이션 경로가 표시됩니다. URL을 클릭하면 브라우저에서 실행중인 애플리케이션을 볼 수 있습니다.
<br><br>
![Application Routes](/img/application_routes_cockpit.png?raw=true)
<br><br>


## 서비스
지금까지 우리는 Java Virtual Machine의 메모리에있는 제품의 표현을 로컬 테스트, 빠른 처리주기에 유용했지만 몇 가지 단점이 있습니다. 응용 프로그램을 다시 시작하면 모든 변경 사항이 손실됩니다. 데이터가 다른 응용 프로그램의 하나의 실행 프로세스보다 다릅니다.이를 해결하고 데이터베이스를 클라우드의 백업 서비스로 추가합시다. 코드를 변경하지 않고도 그렇게 할 수 있습니다. 우리는 서비스 인스턴스를 생성하여 응용 프로그램에 바인드하고 응용 프로그램을 복원하여 데이터베이스 사용을 시작해야합니다. 아래 단계를 따르십시오.
1. 사용자가 사용할 수 있는 서비스 목록 표시 서비스 목록, 서비스 ```cf marketplace``` 계획에 대한 간단한 설명 및 정보가 표시되어야합니다.
2. 예제에서는 PostgreSQL을 사용할 것입니다. 구체적인 서비스에 대한 자세한 정보를 보려면 `cf marketplace -s SERVICE` 명령어를 사용하십시오.

  ```
  cf marketplace -s postgresql
  ```

3. 응용 프로그램에서 서비스를 사용하려면 먼저 서비스 인스턴스를 만들어야합니다.
CF CLI에서 `cf create-service SERVICE PLAN SERVICE_INSTANCE`를 사용하여 PostgreSQL 서비스 인스턴스를 만듭니다.

  ```
  cf create-service postgresql v9.6-dev postgres
  ```

4. 새로 생성 된 postgresql 서비스가 나타납니다. 명령 프롬프트에 다음을 입력하십시오.

  ```
  cf services
  ```
당신의 space 내의 서비스 목록을 확인한다.

5. 마지막 단계는 서비스 인스턴스를 응용 프로그램에 바인딩하는 것입니다.
CF CLI에서 `cf bind-service APP_NAME SERVICE_INSTANCE`을 사용합니다.

```
 cf bind-service product-list postgres
```
6. 이제는 응용 프로그램 환경이 업데이트되도록 백업 서비스를 이미 확보 했으므로 응용 프로그램이 PostgreSQL을 사용하여 지속성을 유지할 수 있도록 응용 프로그램을 다시 준비합니다.
CF CLI에서 `cf restage APP_NAME`을 사용합니다.

  ```
  cf restage product-list
  ```

이 서비스를 CF CLI에서 ```cf services``` 명령어로 확인해보면, PostgreSQL 인스턴스가 제품 목록 응용 프로그램에 포함되어있는 것을 볼 수 있습니다.

PostgreSQL을 사용하기 때문에 차이가 없을지라도 다시 한번 응용 프로그램을 다시 요청할 수 있습니다.

:bulb: **Note** 응용 프로그램에 서비스를 바인딩하는 대신 응용 프로그램 Manifest를 사용하는 것이 좋습니다. manifest.yml을 사용하여 서비스를 바인딩 할 때 환경을 업데이트하고 응용 프로그램이 서비스를 사용할 수 있도록 응용 프로그램을 다시 올려야합니다. 예를 들어 샘플을 추가하려면 스니펫을 추가 한 다음 cf를 입력해 응용 프로그램을 다시 올립니다.

```Config
services:
 - SERVICE_INSTANCE_NAME
```

보통 이 시점에서 서비스 인스턴스를 사용하고 응용 프로그램 코드를 변경해야합니다. 서비스 인스턴스에 액세스하는 방법에 대한 정보는 응용 프로그램 환경에서 환경 변수 VCAP_SERVICES로 사용할 수 있습니다. 우리의 경우 SpringBoot 프레임 워크는 wired PostgreSQL 서비스를 감지하고 이에 대한 작업을 시작하는 메커니즘을 제공하므로 응용 프로그램 코드를 변경할 필요가 없습니다.
