# 스케일 조정

## 예상 시간

:clock4: 20 분

## 목표

이 연습에서는 응용 프로그램의 스케일 조정 방법을 배웁니다.

# Exercise description

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

```cf scale APP_NAME -k DISK``` 명령어는 응용 프로그램의 모든 인스턴스에 적용되는 디스크 공간 제한을 변경하는 데 사용 합니다. DISK는 정수 여야하며 메가 바이트의 경우 M 또는 기가 바이트의 경우 G입니다.

Use ```cf scale APP_NAME -m MEMORY``` 명령어는 응용 프로그램의 모든 인스턴스에 적용되는 메모리 제한을 변경하는 데 사용 합니다. MEMORY는 정수 여야하며 그 뒤에 메가 바이트의 경우 M 또는 기가 바이트의 경우 G가옵니다.

- 우리는 애플리케이션을 축소하고 256MB의 적은 메모리로 샘플 응용 프로그램을 실행하려고합니다.
```
cf scale APP_NAME -m 712M
```
CF CLI에서 이 명령을 실행하면 응용 프로그램이 다시 시작됩니다 (콘솔에서 "예"로 확인하는 것이 좋습니다). 그런 다음 응용 프로그램이 다시 실행되면 조종석에서 응용 프로그램보기를 새로 고치고 712MB 메모리를 사용하는지 확인할 수 있습니다. 콘솔에서도 볼 수 있습니다.
```
cf app APP-NAME
```

### Cockpit
We can do the same operation vertically scaling the applications using cockpit.

Navigate to the the dedicated view for the running application. In the Overview - Quota Information section you see the memory quota and disk quota for your application. You can change this with **Change Quota** button.
<br><br>
![Change Quota](/img/cockpit_change_quota.png?raw=true)
<br><br>
<br><br>
![Change Quota Pop-up](/img/cockpit_change_quota_popup.png?raw=true)
<br><br>

Specify less disk space e.g. 512 MB and click **Save** button. You can check the changed size of disk the application uses.
<br><br>
![Check disc quota](/img/cockpit_scale_disc.png?raw=true)
<br><br>

In case the change is not reflected, you have to manually restart the application for the change to take effect - you can do it with the **Restart** button.
<br><br>
![Restart](/img/cockpit_restart.png?raw=true)
<br><br>

## Horizontal scale

Horizontally scaling an application creates or destroys application instances. Incoming requests to the application are automatically load balanced across all application instances, and each instance handles tasks in parallel with every other instance. Adding more instances allows your application to handle increased traffic and demand.

### CF CLI
Use ```cf scale APP_NAME -i INSTANCES``` to horizontally scale your application. This will lead to increased or decreased number of instances of your application to match INSTANCES.
- We can scale up horizontally the sample application and have 2 instances running. In the command line:
```
cf scale APP_NAME -i 2
```

Executing the command to list information for the application will already show you one running instance and one starting:
```
cf app APP_NAME
```
<br><br>
![Restart](/img/scaling_console.png?raw=true)
<br><br>

### Cockpit
You can also scale horizontally via Cockpit. Navigate in cockpit to the sample application view. You should see in the instances section the two running instances. On the top there are **+ Instance** and **- Instance** buttons. Click on **- Instance** which should result in stopping one of the running instances of the app.
<br><br>
![Restart](/img/cockpit_scaling.png?raw=true)
<br><br>

## Auto-scale

We explored the manual options for scaling up and down an application using the ```cf scale``` command or via Cockpit. The drawback using these is that only if you observe that the performance of your application has degraded, you perform this activity. Your application will crash if action is not taken at the appropriate time. So  it's sometimes better to have a scaling option that is triggered automatically when needed and prevents the application from crashing.

You will learn now how to scale your application up or down automatically based on user defined scaling policies to meet changes in traffic or demand using the SAP Cloud Platform Application Autoscaler service.

### Generate load on the application
We will explore the automatic scaling based on increased memory usage, so that's why we will prepare a simulation that will increase the memory usage of our sample product list application.
- Open **class Controller** and add the following **API endpoint**. This will create a high number of unwanted objects thereby increasing the memory usage
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
Next, let us add an action to trigger the API which will increase memory usage.
- Navigate to `src/main/resources/index.html` and add the below snippet right below the comment ```Insert the code to increase the memory of the application```
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

Now, let us build the project and deploy the application
- Right click on the project in the Project Explorer view -> Run as -> Maven build
- Go to command prompt and run the command ```cf push```
- Launch the application in the browser
- You will see a new action button **Scale Up**
- Check the memory usage of the application running the command ```cf app <app_name>``` (or in cockpit application view)
- Click on the action button **Scale Up**
- Check the memory usage again after few seconds, run the command ```cf app <app_name>``` (or in cockpit application view)

Now that we have prepared the application to scale up the memory usage, let us see how to use the service, ```Application Autoscaler``` to automatically scale the application when the memory usage increases beyond a threshold.

### Create an instance of the service and define scaling rules
You can do this not only with the CF CLI but also via cockpit. Navigate to you trial space - usually the name of the space is **dev**. In the left-hand navigation menu select **Service Marketplace**. You should see **autoscaler** - click on it.  
<br><br>
![Autoscaler in Marketplace](/img/marketplace_autoscaler.png?raw=true)
<br><br>

Now in the left-hand navigation menu select **Instances**.
Click on the **New Instance** button - a pop-up will appear.
<br><br>
![Autoscaler in Marketplace](/img/autoscaler_newinstance.png?raw=true)
<br><br>

In the first step you have to select a service plan. For trial accounts only lite is available, so click on the **Next** button.
<br><br>
![Autoscaler in Marketplace](/img/create_instance_cockpit_1.png?raw=true)
<br><br>

You can skip this step as you can specify the parameters for scaling while binding the application. So click on the **Next** button.
<br><br>
![Autoscaler in Marketplace](/img/create_instance_cockpit_2.png?raw=true)
<br><br>

Skip this step as well, so, So click on the **Next** button.
<br><br>
![Autoscaler in Marketplace](/img/create_instance_cockpit_3.png?raw=true)
<br><br>

As a last step choose a service instance name e.g. myautoscaler and click **Finish** button.
<br><br>
![Autoscaler in Marketplace](/img/create_instance_cockpit_4.png?raw=true)
<br><br>

This should result in Application Autoscaler instance created.

### Bind the application to the service instance
Once the service instance is created you will be in the App Autoscaler instances view. Click on the instance you just created:
<br><br>
![Autoscaler in Marketplace](/img/bind_instance_cockpit_1.png?raw=true)
<br><br>

On the instances page, click on the **Bind Instance** button:
<br><br>
![Autoscaler in Marketplace](/img/bind_instance_cockpit_1.1.png?raw=true)
<br><br>

The below pop-up opens:
<br><br>
![Autoscaler in Marketplace](/img/bind_instance_cockpit_2.png?raw=true)
<br><br>

Select the sample application from the drop-down Application list.
<br><br>
![Autoscaler in Marketplace](/img/bind_instance_cockpit_3.png?raw=true)
<br><br>

Then you should specify parameters for scaling. The rules to scale an application instance are defined in a JSON format. Usually you will create a JSON file and provide it as configuration. For our sample you can copy and paste the snippet below in the big text field:
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
The threshold value indicates the service to scale up the instance by 1 [adjustment: +1] when the memory goes higher than the threshold value. Adjust this value as required by your application.
You can similarly add another scaling rule to scale down when the memory comes down a defined threshold value.

In the pop-up, choose Specify Parameters. In the Enter Parameters text area, copy the above json. Click on **Save** button
<br><br>
![Autoscaler in Marketplace](/img/bind_instance_cockpit_4.png?raw=true)
<br><br>

Now, let's see how `App Autoscaler` helps the application to scale automatically based on application metrics
1. Restart the application to reset the memory usage (you can navigate in cockpit to `Application Overview` page and click **Restart** button)
2. Launch the application in the browser
3. Check the memory of the application using CF CLI command, `cf app <app_name>` or in Cockpit in the `Application Overview` page
4. Click on the **Scale Up** button
5. Check the memory of the application using CF CLI command, `cf app <app_name>` or in Cockpit in the `Application Overview` page after few seconds.
6. It should have increased, ensure that it is higher than the `threshold` value defined in the scaling policy. If not, repeat steps 3 and 4
7. When memory usage is above the `threshold` value for time higher than the `breach_duration` as specified in the scaling policy, the `App Autoscaler` will scale up the instance of the application by 1
8. Check the number of instances of the application using CF CLI command, `cf app <app_name>` or in Cockpit in the `Application Overview` page
<br><br>
![Autoscaler Starting new app instance](/img/autoscaler_starting_new.png?raw=true)
<br><br>
