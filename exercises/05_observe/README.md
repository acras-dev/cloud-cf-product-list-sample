# SAP Cloud Platform Cloud Foundry 환경에서 응용 프로그램 관측

## 예상 시간

:clock4: 10 분

## 목표

이 연습에서는 Cloud Foundry Environment의 평가판 계정에서 실행되는 응용 프로그램 및 서비스에 대한 추가 정보, 기본적인 모니터링 메트릭 및 로그를 얻는 방법을 배우게됩니다.

이 모든 연습을 위해 두 도구 (CF CLI 및 조종석)를 통해 액세스 할 수있는 정보가 있습니다. 아래에서 CF CLI에서 실행해야하는 명령과 정보를 얻기위한 콕 피트 메뉴의 각 탐색을 찾을 수 있습니다.

# 연습과정 설명

Kibana에서 로그를 볼 수 있으려면 서비스 application-logs가 앱에 바인딩되어 있어야합니다.

CLI를 사용하여 응용 프로그램 인스턴스를 만들고 바인딩하십시오.

```
cf marketplace
cf create-service application-logs lite myapplogs
cf services
cf bind-service product-list myapplogs
cf restage product-list
```

## 응용 프로그램 나열
대상 공간에 있는 모든 응용 프로그램의 기본 정보를 나열합니다.

### CF CLI
```
cf apps
```
### 조종석
Start Cloud Foundry Trial의 일환으로 생성 된 평가판 Space으로 이동 한 다음 왼쪽 탐색 메뉴에서 **Applications** 을 클릭하십시오 . 

<br><br>
![List Applications](/img/running_app_cockpit.png?raw=true)
<br><br>

## 서비스 목록
대상 공간에 작성된 서비스 인스턴스 및 바인딩 나열

### CF CLI
```
cf services
```
### 조종석
왼쪽 탐색 메뉴에는 services 메뉴가 있으며 **Service Instances** 를 선택하면 공간에 작성된 모든 인스턴스와 기본 인스턴스 정보가 표시됩니다. 

<br><br>
![List Services](/img/cockpit_services.png?raw=true)
<br><br>

## 로그
응용 프로그램에 대한 로그 표시합니다.

### CF CLI
- Tail logs for an app `cf logs APP_NAME`
```
cf logs product-list
```

:bulb: **Note:** 첫 번째 명령에서 로그를 볼 수없는 경우에 대비하여 응용 프로그램을 요청하십시오..

:bulb: **Note:** 콘솔에서 로그 실시간 스트림 모드를 종료하려면 **Ctrl + C**를 클릭하십시오 

- 응용 프로그램의 최근 로그 표시 명령어 `cf logs APP_NAME`
```
cf logs product-list --recent
```
### 조종석
Cockpit에서 응용 프로그램 로그를 보려면 공간의 응용 프로그램 목록으로 다시 이동하여 푸시 한 응용 프로그램을 선택하고 시작합니다 (응용 프로그램 이름 클릭). 그런 다음 왼쪽 탐색 메뉴에서 **Logs**를 클릭 하면 최근 응용 프로그램 로그가 표시된 테이블이 표시됩니다. 응용 프로그램에 둘 이상의 응용 프로그램 인스턴스가 실행중인 경우 로그를 볼 인스턴스에 대한 드롭 다운을 선택할 수 있습니다. 
<br><br>
![Logs in cockpit](/img/cockpit_logs.png?raw=true)
<br><br>

## 측정항목
CPU, 메모리 및 디스크 사용에 대한 정보를 포함하여 응용 프로그램의 상태 및 상태를 표시합니다.

### CF CLI
콘솔에서 명령 `cf app APP_NAME`을 사용 하여 응용 프로그램에 대한 메트릭을 볼 수 있습니다 .
```
cf app product-list
```
### 조종석
왼쪽 탐색 메뉴의 구체적인 응용 프로그램보기에 이미 있는 경우 **Overview**를 선택 하면 실행중인 응용 프로그램과 유사한 정보가 표시됩니다. 

<br><br>
![Metrics in cockpit](/img/cockpit_overview_app.png?raw=true)
<br><br>

## 이벤트
최근 응용 프로그램 이벤트 (e.g. create, re-stage, update, etc.), 발생한시기 및 누가 실행했는지 보여줍니다.

### CF CLI
```
cf events APP_NAME
```
### 조종석
조종실에 이벤트 표시 - 전용 응용 프로그램보기에서 왼쪽 탐색 **Events**를 선택하십시오 . 
<br><br>
![Metrics in cockpit](/img/cockpit_events.png?raw=true)
<br><br>

## 환경 변수

### 환경 변수 접근
환경 변수를 나열하려면 CF CLI 명령을 사용할 수 있습니다 `cf env APP_NAME`
```
cf env product-list
```
아니면 전용 응용 프로그램보기로 이동하여 왼쪽 탐색 **Environment Variables**에서 선택하여 조종석 또는 조종석을 탐색 할 수 있습니다. 
<br><br>
![Environment variables](/img/cockpit_env_vars.png?raw=true)
<br><br>

다음 예제는 프로그램에서 Java, NodeJS 및 Ruby 샘플과 같은 환경 변수에 프로그램 방식으로 액세스하는 방법을 보여줍니다.

```java
System.getenv("VCAP_SERVICES");
```
```javascript
process.env.VCAP_SERVICES
```
```ruby
ENV['VCAP_SERVICES']
```
### 환경 변수 설정
응용 프로그램 특정 환경 변수는 응용 프로그램 개발자가 CF CLI를 사용하여 설정할 수 있습니다.
```
cf set-env APP_NAME VARIABLE_NAME VARIABLE_VALUE
```
또는 매니페스트에 직접 작성합니다.
```
---
  ...
  env:
    VARIABLE_NAME_1: VARIABLE_VALUE_1
    VARIABLE_NAME_2: VARIABLE_VALUE_2
```
또는 조종석에서 변경합니다.
<br><br>
![Metrics in cockpit](/img/user_provided_vars.png?raw=true)
<br><br>

### 시스템 환경 변수
Cloud Foundry가 응용 프로그램 컨테이너에서 사용할 수있는 환경 변수입니다. 전체 목록은 다음과 같습니다.
- `CF_INSTANCE_ADDR`
- `CF_INSTANCE_GUID`
- `CF_INSTANCE_INDEX`
- `CF_INSTANCE_IP`
- `CF_INSTANCE_INTERNAL_IP`
- `CF_INSTANCE_PORT`
- `CF_INSTANCE_PORTS`
- `HOME`
- `MEMORY_LIMIT`
- `PORT`
- `PWD`
- `TMPDIR`
- `USER`
- `VCAP_APP_HOST`
- `VCAP_APP_PORT`
- `VCAP_APPLICATION`
- `VCAP_SERVICES`

마지막으로 `VCAP_SERVICES`가 중요합니다. 서비스가 바인드 가능한 서비스를 사용하는 경우 Cloud Foundry가 서비스를 바인드 한 후 응용 프로그램을 복원 할 때 연결 세부 정보를 추가하기 때문입니다. 다음 예제는 postgresql 서비스가 바인드 된 응용 프로그램에서 가져온 것입니다.

## 상태 점검
응용 프로그램 상태 검사는 실행중인 Cloud Foundry 응용 프로그램의 상태를 지속적으로 확인하는 모니터링 프로세스입니다. Cloud Foundry에서 기본 시간 초과는 60 초이고 구성 가능한 최대 시간 제한은 180 초입니다. Cloud Foundry에서 사용할 수있는 상태 확인에는 세 가지 유형이 있습니다.

* `http`: `http` 상태 검사는 구성된 HTTP endpoint에 대한 GET 요청을 수행합니다. `HTTP 200` 응답을 받으면 앱이 정상적으로 선언됩니다. 구성된 endpoint는 1 초 이내에 응답해야 건강하다고 간주됩니다.
* `port`: 아무것도 지정하지 않으면 이것이 기본 검사입니다. 그것은 포트 또는 응용 프로그램에 대해 구성된 포트 (기본 8080)에 TCP 연결을합니다. TCP 연결은 정상적인 것으로 간주 되려면 1 초 내에 설정되어야합니다.
* `process`: Diego는 앱용으로 선언 된 프로세스가 계속 실행되도록합니다. 프로세스가 종료되면 Diego는 앱 인스턴스를 중지하고 삭제합니다. 상태 확인은 다음과 같이 지정할 수 있습니다.
  ```
  cf push APP_NAME -u HEALTH-CHECK-TYPE -t HEALTH-CHECK-TIMEOUT
  ```
  또는 직접 매니페스트 안에

  ```
  ---
    ...
    health-check-type: http
    health-check-http-endpoint: /health
    timeout: 120
  ```

### CF CLI
사용방법:
```
cf set-health-check APP_NAME (process | port | http [--endpoint PATH])
```
예제:
```
cf set-health-check APP_NAME http --endpoint PATH (default to /)
cf set-health-check APP_NAME port
cf set-health-check APP_NAME PROCESS_NAME
```
CF CLI 명령 `cf get-health-check APP_NAME`을 사용하여 주어진 앱의 상태 검사 유형을 표시합니다. Product List 응용 프로그램에 대해 구성된 기본 상태 검사가 무엇인지 확인할 수 있습니다.
```
cf get-health-check product-list
```
