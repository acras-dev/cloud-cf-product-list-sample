# 응용 프로그램 업데이트

## 예상 시간

:clock4: 15 분

## 목표

이 과정에서는 Blue-Green deploy 메커니즘을 사용하여 응용 프로그램을 업데이트하는 방법을 배웁니다. 응용 프로그램이 더 복잡한 경우 다중 대상 응용 프로그램 아카이브를 구축하고 deploy하고 MTA를 사용하여 Blue-Green을 자동으로 deploy함으로써 이익을 얻을 수있는 옵션을 탐색합니다.

# 연습과정 설명
업데이트를 표시하려면 동시에 두 개의 실행중인 응용 프로그램 인스턴스가 필요하며 평가판 계정의 한도내에서 이를 수행하려면 처음에 하나의 샘플 응용 프로그램만 실행하면 됩니다. 다음 업데이트된 버전을 추가합니다.

## Blue-green deployment
Blue-Green deploy는 Blue 및 Green이라는 두 개의 동일한 생산 환경을 실행하여 가동 중지 시간과 위험을 줄이는 기술입니다. Blue가 현재 라이브이고 Green이 유휴 상태라고 가정합시다. 새 버전의 소프트웨어를 준비 할 때 deploy 및 최종 테스트 단계는 실제 환경이 아닌 환경 (예 : Green)에서 수행됩니다. Green에서 소프트웨어를 deploy하고 완전히 테스트 한 후에는 들어오는 모든 요청이 Blue 대신 Green으로 바뀌도록 라우터를 전환합니다. Green은 현재 라이브이며 Blue는 유휴 상태입니다.

<br><br>
![Blue Green 1](/img/blue_green_1.png?raw=true)
<br><br>
![Blue Green 2](/img/blue_green_2.png?raw=true)
<br><br>

이 기술을 사용하면 응용 프로그램 deploy로 인한 가동 중지 시간을 줄이고 위험이 줄어 듭니다. 새 버전의 Green에서 예상치 못한 문제가 발생하면 트래픽을 Blue로 다시 라우팅하여 마지막 버전으로 즉시 롤백 할 수 있습니다.

:bulb: **Note:** 응용 프로그램에서 관계형 데이터베이스를 사용하는 경우 blue-green deployment는 업데이트 도중 Green 데이터베이스와 Blue 데이터베이스간에 불일치가 발생할 수 있습니다. 데이터 무결성을 최대화하려면 역방향 및 정방향 호환성을 위해 단일 데이터베이스를 구성하십시오.

### 실전

업데이트에 대한 변경 준비 - 업데이트로 새 버전의 앱을 쉽게 알아보기 위해 **index.html**을 변경해 보겠습니다. Eclipse 에서 index.html 파일을 열고 제품 목록 페이지가 작성된 원시를 찾고 제품에서 Green 제품으로 제목을 변경하십시오.

```java
// create the list page
			var products = new sap.m.Page("products", {
				title : "Green Products",
				showNavButton : false,
				content : productList
			});
```
2. 응용 프로그램 매니페스트 파일을 열고 응용 프로그램 이름을 changed-product-list로 변경하고 호스트를 원래 응용 프로그램과 다른 다른 것으로 변경합니다. 예 : 접두사를 다시 추가하고 **manifest.yml**을 저장합니다. 변경 후 Maven 빌드 실행 (Run As -> Maven build)
3. 방금 변경 한 manifest.yml 이 있는 샘플 응용 프로그램 루트 폴더에서 콘솔을 열고 새 버전을 다른 이름으로 ```cf push``` 명령을 실행합니다.
4. 응용 프로그램의 두 인스턴스가 실행 중입니다. 모든 트래픽이 전달되는 "productive"이며 다른 URL (route)을 통해 액세스를 테스트 할 수있는 새로운 트래픽입니다. 변경된 버전이 다른 URL에서 접근하는 것으로 예상되는대로 작동하는지, 제목이 **Green Products** 인지 확인합니다.
<br><br>![New app version](/img/changed_app_UI.png?raw=true)
<br><br>
5. 이제 두 앱이 모두 실행되고 있습니다. 들어오는 모든 요청이 새 앱 버전 (Green)과 이전 버전 (Blue)으로 바뀌도록 라우터를 전환하십시오. cf map-route 명령을 사용하여 원본 URL 경로를 Green ```cf map-route Green_APP_NAME DOMAIN -n Blue_APP_HOSTNAME``` 명령어로 어플리케이션에 매핑하여 이를 수행하십시오.
6. 몇 초 내에 cfmap-route 명령을 실행하면 CF 라우터는 Blue 및 Green 버전의 응용 프로그램간에 원래의 생산 URL에 대한 트래픽 부하 분산을 시작합니다.
7. 경로를 Blue 버전으로 매핑 해제합니다. Green 버전이 예상대로 실행되는지 확인한 후 아래 명령어로 Blue 버전으로 라우팅 되지 않도록 변경합니다.
```
cf unmap-route Blue_APP_NAME DOMAIN -n HOSTNAME
```
명령 후 CF 라우터는 Blue 버전으로 트래픽 전송을 중단합니다. 이제는 생산적인 도메인의 모든 트래픽이 Green 버전으로 전송됩니다.

Green 버전의 임시 경로를 제거하려면 cf unmap-route를 사용할 수 있습니다. cf delete-route를 사용하여 경로를 삭제하거나 나중에 사용할 수 있도록 예약 할 수 있습니다. Blue을 폐기하거나 변경 사항을 롤백해야 할 경우를 대비하여 유지할 수도 있습니다.

## MTA 및 자동화 Blue-Green deployment

다음 연습을 계속 진행하려면 제한된 리소스가 있으므로 모든 응용 프로그램 및 서비스 인스턴스를 삭제하여 평가판 계정을 정리해야합니다.

### MTA 아카이브 만들기
deploy 준비 다중 대상 응용 프로그램 (MTA)을 구축하려면 deployment descriptor (mta.yaml)로 MTA 모듈을 정의한 다음 MTA 보관 파일 (MTAR)을 패키지화 해야합니다. [MTA Archive Builder tool](https://help.sap.com/viewer/58746c584026430a890170ac4d87d03b/HANA%202.0%20SPS%2002/en-US/ba7dd5a47b7a4858a652d15f9673c28d.html)를 사용합니다.

1. mta.jar (파일 위치)를 사용할 수있는 MTA 보관 도구 작성 도구가 이미 있습니다. 응용 프로그램의 루트 폴더 옆에 복사하십시오.
2. 응용 프로그램의 루트 폴더에 빈 mta.yaml 파일을 만듭니다.
3. 이제 응용 프로그램 외에도 mta.yaml을 통해 응용 프로그램에 바인딩 할 서비스 인스턴스를 지정할 수 있습니다.

```Configuration
_schema-version: "2.0.0"
ID: product-list
version: 1.0.0

modules:
  - name: product-list
    type: java
    path: .
    parameters:
      memory: 812M
      buildpack: java_buildpack
    properties:
      SPRING_PROFILES_ACTIVE: cloud
    provides:
      - name: product-list
        properties:
          url: "${default-url}"
    requires:
      - name: postgres
      - name: logging
    build-parameters:
      build-result: target/*.jar

resources:
  - name: postgres
    type: postgresql
  - name: logging
    type: application-logs
    parameters:
      service-plan: lite
```
편집 후 파일을 **Save**하십시오.
4. MTAR 빌드 MTA 보관 빌더 도구를 실행해야 합니다. 샘플 응용 프로그램의 루트 폴더에서 터미널을 엽니다. java -jar [path to mta.jar] --build-target = CF build 명령어를 실행합니다. 
```
java -jar ../mta.jar --build-target=CF build
```

결과적으로 응용 프로그램의 루트 폴더에 mtar 아카이브가 생성됩니다. 필자의 경우 이름은 product-list.mtar이지만 로컬 환경에 따라 다를 수 있습니다. 명령의 마지막 부분에서 볼 수도 있고 앱의 루트 폴더에서 확인할 수도 있습니다.

### MTA deploy

MTAR을 deploy하려면 MTA CF CLI 플러그인이 필요합니다. MTA CF CLI 플러그인을 [다운로드](https://tools.hana.ondemand.com/#cloud) 하십시오.

선행조건: 커맨드 라인을 통해 Cloud Foundry Environment에 로그인합니다.

1. MTAR product-list.mtar deploy

	```
	cf deploy product-list.mtar
	```
2. 과적으로 응용 프로그램이 전개되고 서비스 인스턴스가 작성됩니다. CF CLI에서 명령을 통해 mta가 deploy되었는지 확인합니다.

	```
	cf mtas
	```
```cf apps``` 명령어로 응용 프로그램이 실행되고 있는지 확인할 수 있습니다.

### 자동화 Blue-Green deployment
응용 프로그램을 요청하십시오 - 이제 Green Products 제목 버전이 실행 중입니다.

업데이트에 대한 변경 준비 - 업데이트로 새 버전의 응용 프로그램을 쉽게 알 수 있도록 index.html을 다시 변경해 보겠습니다. Eclipse 에서 **index.html** 파일을 열고 제품 목록 페이지가 작성된 원시를 찾고 제품에서 Green 제품으로 제목을 변경하십시오.

```java
// create the list page
			var products = new sap.m.Page("products", {
				title : "MTA Blue/Green Products",
				showNavButton : false,
				content : productList
			});
```
1. MTAR을 다시 빌드하여 deploy 용으로 패키지 된 새 버전의 응용 프로그램을 가져옵니다.

	```
	java -jar ../mta.jar --build-target=CF build
	```
2. 아래 Blue/Green 명령을 사용하여 MTAR deploy합니다.

	```
	cf bg-deploy MTAR_FILE
	```
명령이 성공하면 아래와 비슷한 메세지가 나옵니다.

```
Application "product-list-Green" started and available at "Here you will find URL of the Green application version"
```
브라우저에서 열어 제목 변경이 적용되었는지 확인합니다.

위에서 수행 한 작업은 이미 MTA의 버전이 deploy되었는지 확인하고 새 버전을 deploy하고 유효성을 검사 할 수 있도록 임시 경로를 등록한 것입니다. 테스트가 성공적이면 새 앱 버전을 공식 경로에 바인딩 할 수 있습니다. ```cf map-route Green_APP_NAME DOMAIN -n Blue_APP_HOSTNAME```
