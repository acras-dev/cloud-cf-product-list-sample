# 응용 프로그램 로깅 서비스와 통합

## 예상 시간

:clock4: 15 분

## 목표

이 연습에서는 응용 프로그램이 SAP Cloud Platform 응용 프로그램 로깅 서비스와 통합되는 방법과 Kobana 대시 보드의 응용 프로그램 로그를 Cloud Foundry 구성 요소 로그와 함께 시각화하는 방법을 배우게됩니다.


# 연습과정 설명


## 로깅
먼저 응용 프로그램의 소스 코드를 살펴 보겠습니다. 이미 SAP 라이브러리(Cloud Foundry에 대한 로깅 지원 : https://github.com/SAP/cf-java-logging-support) 사용됩니다. pom.xml에는 필요한 종속성이 선언되고, 구성 (logback.xml) 및 구현 (ConfigLogging ) 사용할 수 있습니다.

* 이클립스 실행후 my-product-list 응용 프로그램의 클래스 `Controller.java`를 엽니다.
  - 클래스 내부에 `Logger` 가 있습니다.
```java
private static final Logger logger = LoggerFactory.getLogger(Controller.class);
```

  - 메소드 `getProductByName` 다음 로그 항목을 씁니다.
  ```java
  logger.info("***First - Retrieving details for '{}'.", name);
  logger.info("***Second - Retrieving details for '{}'.", name);
  ```
* 응용 프로그램 로그 서비스는 이미 생성되어 product-list에 바인딩되어 있습니다.
* 로그를 생성하기 전에 logback.xml 파일을 변경하십시오.
* 이클립스에서 logback.xml을 열고 `<appender-ref ref="STDOUT-JSON" />`에서 `<appender-ref ref="STDOUT" />`로 변경하십시오.
* 이클립스에서 - 해당 project 오른쪽 클릭 -> Run As... -> Maven Build
* 터미널에서 응용 프로그램의 루트폴더로 이동한 다음 `cf push product-list` 입력
<br><br>

## 로그 생성
* 로그를 생성하려면 URL을 통해 응용 프로그램을 요청하는것으로 충분합니다.
![Application Routes](/img/application_routes_cockpit.png?raw=true)
<br><br>
* 브라우저에서 응용 프로그램 로그 요청을 생성할때는 아래 상대 경로를 참고하십시오.
`YOUR_APPLICATION_URL/productsByParam?name=Notebook Basic 15`
* 조종실에서 내 product-list 응용 프로그램에 대한 로그를 탐색합니다 (왼쪽 탐색 패널의 Logs 탭 클릭). 다음 아래와 같이 **Open Log Analysis** 링크를 클릭하십시오.
<br><br>
![Application Log Analysis](/img/cockpit_open_log_analysis.png?raw=true)
<br><br>
* 브라우저에서 다음 URL을 열어야합니다 (EU10 지역에서 실행되는 응용 프로그램의 경우 URL은 us10 대신 eu10입니다). https://logs.cf.us10.hana.ondemand.com
* 조종석에 사용하고 평가판 계정을 만든 전자 메일 및 암호로 로그인하십시오.
* Kibana의 product-list 앱 선택
<br><br>
![Kibana select app](/img/kibana_product_list_app.png?raw=true)
<br><br>
* `<Press>` 요청 및 로그
<br><br>
![Kibana requests and logs](/img/kibana_requests_logs.png?raw=true)
<br><br>
* 요청 및 응용 프로그램 로그 표시됩니다.
* 응용 프로그램 로그의 msg 필드가 이상하게 보입니다.
  * 관련이 없다는 것을 보여줍니다. 애플리케이션 로그에는 상관 ID가 없습니다.
  * 따라서 어떤 응용 프로그램 로그가 어떤 요청에 속해 있는지 알기가 어렵습니다. 
  <br><br>
  ![Kibana Message](/img/kibana_msg_no_correlationid.png?raw=tru)
  <br><br>

:bulb:**Note:** 경우에 따라 Kibana에 로그가 표시되지 않을 수있습니다. 기본적으로 지난 15 분간의 로그만 볼 수 있기 때문입니다. 오른쪽 상단 모서리에서이 간격을 변경할 수 있습니다. 
<br><br>
![Kibana recent logs](/img/kibana_recent_logs.png?raw=true)
<br><br>

* In Eclipse open **logback.xml**. Change `<appender-ref ref="STDOUT" />` to `<appender-ref ref="STDOUT-JSON" />`
* In Eclipse build the project - Right click on the project -> Run As... -> Maven Build
* Go into the root folder of the application via command line and push it with `cf push product-list`
* Once the application is up and running in your Cloud Foundry trial account, go to cockpit, navigate to the application and request the application route in a browser adding the relative path that generates logs: `YOUR_APP_URL/productsByParam?name=Notebook%20Basic%2015`
* If Kibana is still open, you can just refresh the tab to show latest logs. If you closed it, go again to cockpit, in the application view, click on Logs tab in the left hand navigation and click the Kibana link (click on **Open Log Analysis**)
  * You can see now that the msg field in applications logs displays now the beautiful formatted message
  <br><br>
  ![Kibana logs format](/img/kibana_logs_format.png?raw=true)
  <br><br>
  * You can also see that the Application Logs messages are now linked via the Correlation Id to the Requests:
  <br><br>
  ![Kibana correlatioIDs](/img/kibana_correlationIDs.png?raw=true)
  <br><br>

:bulb: If you want to learn more about Application Logging in SAP Cloud Platform Cloud Foundry Environment take a look at the official documentation:

:link:[Application Logging overview in official documentation](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/68454d44ad41458788959485a24305e2.html)
