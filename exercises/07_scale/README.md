# 스케일 조정

## 예상 시간

:clock4: 20 분

## 목표

이 연습에서는 응용 프로그램의 스케일 조정 방법을 배웁니다.

# 연습과정 설명

사용자로드 또는 응용 프로그램에서 수행하는 작업의 수와 특성과 같은 요소는 응용 프로그램에서 사용하는 디스크 공간과 메모리를 변경할 수 있습니다. 많은 응용 프로그램에서 사용 가능한 디스크 공간, 메모리를 늘리거나 더 많은 인스턴스 (CPU)를 실행하면 전반적인 성능을 향상시킬 수 있습니다. 마찬가지로 응용 프로그램의 추가 인스턴스를 실행하면 응용 프로그램이 사용자로드 및 동시 요청의 증가를 처리 할 수 있습니다. 이를 응용 프로그램의 스케일 조정이라고 합니다.

CF CLI 또는 조종석을 통해 수동으로 트래픽 또는 수요의 변화에 맞게 응용 프로그램을 위 / 아래로 확장하고 SAP Cloud Platform Application Autoscaler 서비스를 자동으로 사용할 수 있습니다.


## 스케일링을 위한 준비

계정에서 이미 실행중인 응용 프로그램이 두 개 이상인 경우에는 크기 조정 옵션을 시도하기 위해 쿼터가 남도록 조금씩 정리해야합니다. 필요한 것은 계정에서 실행되는 SpringBoot Product List 응용 프로그램의 한 인스턴스입니다.
- 터미널 열기
- 대상 Space에 있는 모든 응용 프로그램 나열
```
cf apps
```
- 해당 Space에서 실행중인 하나의 Product List 앱 이외에 더 많은 응용 프로그램이있는 경우 다음 연습을 위해 메모리 할당량을 비우도록 각 응용 프로그램에 대해 중지 명령을 실행합니다.
```
cf stop APP_NAME
```
:bulb:**Note:** 다음 연습을 계속하기 전에 계정에서 Product List 응용 프로그램 인스턴스 하나를 실행해야합니다. 마스터 브랜치에서 샘플을 복제하고 올릴수 있습니다.

## 수직 스케일

응용 프로그램을 세로로 확장하면 Cloud Foundry가 응용 프로그램의 모든 인스턴스에 적용하는 디스크 공간 제한 또는 메모리 제한이 변경됩니다.

### CF CLI

```cf scale APP_NAME -k DISK``` 명령어는 응용 프로그램의 모든 인스턴스에 적용되는 디스크 제한을 변경하는 데 사용 합니다. DISK 크기값은 정수며 뒤에 메가 바이트의 경우 M, 기가 바이트의 경우 G를 사용합니다.

```cf scale APP_NAME -m MEMORY``` 명령어는 응용 프로그램의 모든 인스턴스에 적용되는 메모리 제한을 변경하는 데 사용 합니다. MEMORY 크기값은 정수며 뒤에 메가 바이트의 경우 M, 기가 바이트의 경우 G를 사용합니다.

- 우리는 애플리케이션을 축소하고 256MB의 적은 메모리로 샘플 응용 프로그램을 실행하려고합니다.
```
cf scale APP_NAME -m 712M
```
CF CLI에서 이 명령을 실행하면 응용 프로그램이 다시 시작됩니다 (콘솔에서 "예"로 확인하는 것이 좋습니다). 그런 다음 응용 프로그램이 다시 실행되면 조종석에서 응용 프로그램보기를 새로 고치고 712MB 메모리를 사용하는지 확인할 수 있습니다. 콘솔에서도 볼 수 있습니다.
```
cf app APP-NAME
```

### 조종석
조종석을 사용하여 응용 프로그램을 수직 확장하는 동일한 작업을 수행 할 수 있습니다.

실행중인 응용 프로그램의 전용보기로 이동합니다. 개요 - 할당량 정보 섹션에는 응용 프로그램의 메모리 할당량과 디스크 할당량이 표시됩니다. **Change Quota** 버튼을 사용하여 이 값을 변경할 수 있습니다.
<br><br>
![Change Quota](/img/cockpit_change_quota.png?raw=true)
<br><br>
<br><br>
![Change Quota Pop-up](/img/cockpit_change_quota_popup.png?raw=true)
<br><br>

원하는 디스크 공간 (예 : 512MB)을 지정하고 **Save** 버튼을 누릅니다. 응용 프로그램에서 사용하는 디스크의 변경된 크기를 확인할 수 있습니다. 
<br><br>
![Check disc quota](/img/cockpit_scale_disc.png?raw=true)
<br><br>

변경 사항이 반영되지 않은 경우 변경 사항을 적용하려면 응용 프로그램을 수동으로 다시 시작해야 합니다. **Restart** 버튼을 사용하여 변경할 수 있습니다.
<br><br>
![Restart](/img/cockpit_restart.png?raw=true)
<br><br>

## 수평 스케일

응용 프로그램을 가로로 확장하면 응용 프로그램 인스턴스가 작성되거나 파기됩니다. 응용 프로그램에 대한 수신 요청은 모든 응용 프로그램 인스턴스에서 자동으로로드 균형 조정되며 각 인스턴스는 다른 모든 인스턴스와 병행하여 작업을 처리합니다. 더 많은 인스턴스를 추가하면 응용 프로그램이 증가 된 트래픽 및 수요를 처리 할 수 있습니다.

### CF CLI
Use ```cf scale APP_NAME -i INSTANCES``` 명령어는 응용 프로그램을 수평으로 확장하는 데 사용 합니다. 이렇게하면 INSTANCES와 일치하는 응용 프로그램의 인스턴스 수가 증가 또는 감소합니다.
- 샘플 응용 프로그램을 수평으로 확장하고 2 개의 인스턴스를 실행시킬 수 있습니다. 터미널에서 아래 명령어를 실행합니다.
```
cf scale APP_NAME -i 2
```

응용 프로그램에 대한 정보를 나열하는 아래 명령을 실행하면 이미 실행중인 인스턴스 하나와 시작중인 인스턴스가 표시됩니다.
```
cf app APP_NAME
```
<br><br>
![Restart](/img/scaling_console.png?raw=true)
<br><br>

### 조종석
조종실을 통해 수평으로도 확장 할 수 있습니다. 조종실에서 샘플 애플리케이션보기로 이동합니다. 인스턴스 섹션에 두 개의 실행중인 인스턴스가 표시되어야합니다. 맨 위에는 **+ Instance** 와 **- Instance** 버튼이 있습니다. 실행중인 앱 인스턴스 중 하나를 중지해야하는 **- Instance** 를 클릭 합니다. 
<br><br>
![Restart](/img/cockpit_scaling.png?raw=true)
<br><br>

## 자동 스케일

우리는 ```cf scale``` 명령을 사용하거나 조종실을 통해 응용 프로그램을 위 / 아래로 조정하기위한 수동 옵션을 살펴 보았습니다 . 이것들을 사용하는 단점은 응용 프로그램의 성능이 저하되었다는 것을 관찰 할 때만이 활동을 수행한다는 것입니다. 적절한 시간에 조치가 취해지지 않으면 애플리케이션이 중단됩니다. 따라서 필요할 때 자동으로 실행되는 확장 옵션을 사용하는 것이 좋으며 응용 프로그램이 충돌하는 것을 방지 할 수 있습니다.

이제 사용자 정의 확장 정책을 기반으로 애플리케이션을 자동으로 위 또는 아래로 확장하여 SAP Cloud Platform Application Autoscaler 서비스를 사용하여 트래픽 또는 수요의 변화를 충족시키는 방법을 학습합니다.

### Generate load on the application
메모리 사용량 증가에 따른 자동 확장 기능을 탐색하므로 샘플 product list 응용 프로그램의 메모리 사용량을 늘릴 시뮬레이션을 준비합니다.
- **class Controller** 를 열고 **API endpoint** 를 추가하십시오. 이렇게하면 많은 수의 원하지않은 객체가 만들어져 메모리 사용량이 증가합니다.
```java
	@GetMapping("/scaleup")
	public String scaleUp() {
		String str = "";
		HashMap<String, Double> h = null;
	    for (int i = 0; i < 100000; i++) {
			h = new HashMap<>();
			String key = new String("key");
			Double value = new Double(100.98);
	        h.put(key, value);
			str = str + h.toString();
	    }
		return "success";
	}
```
Organize imports -> add `import java.util.HashMap;`
다음으로 API를 트리거하는 액션을 추가하여 메모리 사용량을 늘리십시오.
- `src/main/resources/index.html` 경로에서 ```Insert the code to increase the memory of the application``` 를 찾고 그 구문 밑에 아래 스니펫을 추가합니다.
```javascript
/* Adding Toolbar to the UI */
var toolBar = new sap.m.Toolbar();
var memButton = new sap.m.Button({
    title: "Scale Up",
    press: function (oEvent) {
        jQuery.sap.require("sap.m.MessageBox");
        sap.m.MessageBox.show(
            "This will scale up the memory of the application", {
                icon: sap.m.MessageBox.Icon.INFORMATION,
                title: "ScaleUp Application",
                actions: [sap.m.MessageBox.Action.OK],
                onClose: function(oAction) {
                    var oScaleModel = new sap.ui.model.json.JSONModel();
                    var scaleUpUrl = document.URL + "scaleup";
                    oScaleModel.loadData(scaleUpUrl);
                    console.log(scaleUpUrl);
                }
            }
        );
    }
});
memButton.setText("Scale Up");
toolBar.addContent(memButton);
productList.setHeaderToolbar(toolBar);
```

이제 프로젝트를 빌드하고 애플리케이션을 배포합니다.
- 해당 프로젝트 마우스 오른쪽 클릭 -> Run as -> Maven build
- 터미널에서 ```cf push``` 명령어 실행
- 브라우저에서 응용 프로그램 실행
- **Scale Up** 버튼이 보이게 됩니다.
- 응용 프로그램의 메모리 사용량을 ```cf app <app_name>``` 명령어를 사용하거나 조종석의 application view를 통해 확인합니다.
- **Scale Up** 버튼을 클릭합니다.
- 몇 초 후 응용 프로그램의 메모리 사용량을 ```cf app <app_name>``` 명령어를 사용하거나 조종석의 application view를 통해 확인합니다.

이제 메모리 사용량을 확장하는 응용 프로그램을 준비 했으므로, ```Application Autoscaler```을 사용한 메모리 사용량이 임계 값을 초과하여 증가하면 응용 프로그램의 자동 확장을 위한 서비스 사용 방법을 살펴 보겠습니다.

### 서비스의 인스턴스를 만들고 스케일링 규칙을 정의하십시오.
CF CLI뿐만 아니라 조종실을 통해서도 이 작업을 수행 할 수 있습니다. 평가판 Space로 이동한 뒤 왼쪽 탐색 메뉴에서 **Service Marketplace**를 선택하십시오 . **autoscaler**가 보일 것입니다. 
<br><br>
![Autoscaler in Marketplace](/img/marketplace_autoscaler.png?raw=true)
<br><br>

이제 왼쪽 탐색 메뉴에서 **Instances**를 선택하십시오. **New Instance**을 클릭하면 팝업이 뜹니다.
<br><br>
![Autoscaler in Marketplace](/img/autoscaler_newinstance.png?raw=true)
<br><br>

첫 번째 단계에서는 서비스 계획을 선택해야 합니다. 평가판 계정의 경우 lite만 사용할 수 있으므로 **Next** 버튼을 클릭하십시오.
<br><br>
![Autoscaler in Marketplace](/img/create_instance_cockpit_1.png?raw=true)
<br><br>

응용 프로그램을 바인딩하는 동안 크기 조정을 위한 매개 변수를 지정할 수 있으므로 이 단계를 건너 뛸 수 있습니다. **Next** 버튼을 클릭하십시오.
<br><br>
![Autoscaler in Marketplace](/img/create_instance_cockpit_2.png?raw=true)
<br><br>

이 단계도 건너 뛰므로 **Next** 버튼을 클릭하십시오.
<br><br>
![Autoscaler in Marketplace](/img/create_instance_cockpit_3.png?raw=true)
<br><br>

마지막 단계에서 myautoscaler와 같은 서비스 인스턴스 이름을 선택하고 **Finish** 버튼을 누릅니다.
<br><br>
![Autoscaler in Marketplace](/img/create_instance_cockpit_4.png?raw=true)
<br><br>

이로 인해 Application Autoscaler 인스턴스가 만들어집니다.

### 응용 프로그램을 서비스 인스턴스에 바인딩
서비스 인스턴스가 생성되면 App Autoscaler 인스턴스 보기에서 볼 수 있습니다. 방금 생성 한 인스턴스를 클릭하십시오.
<br><br>
![Autoscaler in Marketplace](/img/bind_instance_cockpit_1.png?raw=true)
<br><br>

인스턴스 페이지에서 **Bind Instance** 버튼을 클릭합니다.
<br><br>
![Autoscaler in Marketplace](/img/bind_instance_cockpit_1.1.png?raw=true)
<br><br>

아래처럼 팝업이 열립니다.
<br><br>
![Autoscaler in Marketplace](/img/bind_instance_cockpit_2.png?raw=true)
<br><br>

드롭 다운 응용 프로그램 목록에서 샘플 응용 프로그램을 선택하십시오.
<br><br>
![Autoscaler in Marketplace](/img/bind_instance_cockpit_3.png?raw=true)
<br><br>

다음 스케일링을 위한 매개 변수를 지정해야합니다. 애플리케이션 인스턴스를 확장하는 규칙은 JSON 형식으로 정의됩니다. 일반적으로 JSON 파일을 만들어 구성으로 제공합니다. 샘플의 경우 큰 텍스트 입력란에 스니펫을 복사하여 붙여 넣을 수 있습니다.
```Config
{
    "instance_min_count": 1,
    "instance_max_count": 3,
    "scaling_rules": [
        {
            "metric_type": "memoryused",
            "stat_window_secs": 60,
            "breach_duration_secs": 60,
            "threshold": 400,
            "operator": ">=",
            "cool_down_secs": 60,
            "adjustment": "+1"
        }
    ]
}
```
임계 값은 메모리가 임계 값보다 높으면 인스턴스를 1 [adjustment : +1] 단위로 확장하는 서비스를 나타냅니다. 응용 프로그램에서 필요에 따라이 값을 조정하십시오. 메모리가 정의 된 임계 값 이하로 떨어지면 다른 스케일링 규칙을 추가하여 축소 할 수 있습니다.

팝업에서 매개 변수 지정을 선택하십시오. 매개 변수 입력 텍스트 영역에서 위의 json을 복사합니다. **Save** 버튼을 클릭하십시오.
<br><br>
![Autoscaler in Marketplace](/img/bind_instance_cockpit_4.png?raw=true)
<br><br>

이제  `App Autoscaler`가 응용 프로그램 메트릭을 기반으로 응용 프로그램이 자동으로 확장되는 방법을 알아 보겠습니다.
1. 응용 프로그램을 다시 시작하여 메모리 사용을 재설정하십시오. (조종실에서 Application Overview 페이지로 이동하여 **Restart** 버튼 클릭)
2. 브라우저에서 응용 프로그램 실행
3. CF CLI 명령 `cf app <app_name>` 또는 조종석의 `Application Overview 페이지`에서 응용 프로그램의 메모리를 확인 하십시오.
4. **Scale Up** 버튼을 클릭하십시오.
5. 몇 초 후 CF CLI 명령 `cf app <app_name>` 또는 조종석의 `Application Overview 페이지`에서 응용 프로그램의 메모리를 확인 하십시오.
6. 증가해야하며 확장 정책에 정의 된 값 보다 높아야합니다. 그렇지 않은 경우 3 단계와 4 단계를 반복하십시오.
7. 메모리 사용량이 확장 정책에 지정된 `threshold` 시간보다 높은 시간 동안 값을 초과 하면 응용 프로그램의 인스턴스가 1 씩 증가합니다.
8. CF CLI 명령 `cf app <app_name>` 또는 조종석의 `Application Overview 페이지`에서 응용 프로그램의 인스터스와 메모리를 확인 하십시오.
<br><br>
![Autoscaler Starting new app instance](/img/autoscaler_starting_new.png?raw=true)
<br><br>
