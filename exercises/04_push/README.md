# Exercise 03: Push the application to SAP Cloud Platform Cloud Foundry Environment

## 예상 시간

:clock4: 10 분

## 목표
이 연습에서는 Cloud Foundry Environment 평가판 계정에서 SAP Cloud Platform의 애플리케이션을 푸시 할 수있는 방법을 학습합니다. 우리는 Cloud Foundry Command Line Interface를 사용할 것입니다. 명령 프롬프트를 열고 아래 단계를 수행하십시오.

# 연습문제 설명
## 대상 & 로그인

:bulb: **Note** (Windows) 터미널을 실행하십시오. For that please:
	- 윈도우키 + 'R' 키 누르기
	- 입력창에 ```cmd``` 입력후 엔터

You need to tell the CF CLI which Cloud Foundry you will use. To do this you have to set the API endpoint to the Cloud Controller of the Cloud Foundry region where you created your Cloud Foundry trial using

1. 사용할 Cloud Foundry CLI를 알려줘야 합니다. 이렇게 하려면 아래Cloud Foundry 시험판을 만든 Cloud Foundry 지역의 Cloud Controller에 API endpoint를 설정해야 합니다
```cf api CLOUD_FOUNDRY_API_ENDPOINT```.
 - US10 지역의 API endpoint:
```
cf api https://api.cf.us10.hana.ondemand.com
```
 - EU10 지역의 API endpoint:
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

:bulb: **Note:** SAP Cloud Platform Cloud Foundry Environment의 이전 사용으로 인해 지적한 지역에 둘 이상의 Cloud Foundry 조직과 공간을 생성 한 경우 cf target 명령에서 어느 것을 사용해야하는지 선택하거나 프롬프트가 표시 될 때 선택합니다.
```
cf target -o ORGANIZATION -s SPACE
```

이제 Cloud Foundry 공간에서 작업 할 준비가되었습니다.

## Application Manifest
When you push an application to the Cloud Foundry Environment you can either provide parameters in CF CLI to the push command or define an application deployment descriptor file (manifest.yml) and summarize there the push parameters instead of typing these every time you push. For more information regarding application manifest file you can check the [Cloud Foundry documentation](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)

* Now we will create an application manifest.yml file for our sample Product List application. Go to Eclipse again and in the Project Explorer in Eclipse on the sample Product List app  -> right click -> New -> File. create file and name it **manifest.yml**.

* Now open the file and insert the following snippet adjusting it to your application:

  :bulb: **Note** ```path``` should point to the jar file produced in you project's target folder as an outcome of the Maven build, so maybe you will have to replace the name of the jar with your jar file name.

  :bulb: **Note** ```host``` should be unique as the host namespace is shared with all applications within Cloud Foundry, so add at the end your birth date - day, month and year.

```Configuration
 applications:
 # Application
 - name: product-list
   instances: 1
   memory: 896M
   host: product-list-YOUR-BIRTHDATE-DAY-MONTH-YEAR
   path: target/my-product-list-0.0.1-SNAPSHOT.jar
   buildpack: https://github.com/cloudfoundry/java-buildpack.git#v4.3
```
**Save** the file after editing.

## Push
- At the command prompt, go to the root directory of your SpringBoot application where you just created the manifest.yml file. Type the following command:
```
cf push
```
This triggers the application deployment process using the application manifest.yml you just created. The fist step is uploading the application binaries to the cloud. In the next phase, the so called staging happens which results into a :droplet: droplet. Then the release phase tells the Cloud Foundry how to start the application. Once the command finishes with success you can request the application in a browser - copy and paste the url printed in console.

:bulb:**Note:** In case the *command fails*, read through the error message. It may be you didn't change the parameters in manifest.yml for your local sample properly. Or the quota in your trial account exceeded - if you already used your trial you have to clean-up applications and service instances prior continuing with the exercises. You can do so via cockpit:
- navigate to the list of Applications running in your Space and delete all or stop all running applications;
- go to the Service Instances and delete all.

## Request the application
Now you have the application running in the SAP Cloud Platform Cloud Foundry Environment in your trial account. Open Chrome browser - navigate to SAP Cloud Platform cockpit:
- https://account.hana.ondemand.com/cockpit#/home/overview
- You should land in your user home. If you are not there, click on Home.
- Click on Go to Cloud Foundry Trial button
<br><br>
![GO to trial](/img/go_to_trial_button.png?raw=true)
<br><br>
- You will land in your trial Global Account -> Click on the subaccount (usually the name is trial) -> Click on Spaces in the left hand navigation --> click on the name of your space (it's usually dev)
<br><br>
![GO to space](/img/cockpit_CF_space_org.png?raw=true)
<br><br>
- You are now in your trial space where you can see the running applications - in our case it should be your Product List application that you just pushed to the cloud. You can click on the application name:
<br><br>
![Running applications in cockpit](/img/running_app_cockpit.png?raw=true)
<br><br>
A dedicated application view will open where you will see the Application Routes - click on the URL to see your application running in a browser.
<br><br>
![Application Routes](/img/application_routes_cockpit.png?raw=true)
<br><br>


## Services
So far we have used representation of the products in the memory of the Java Virtual Machine which is good for local testing, fast turnaround cycles, however it has some drawbacks – if one restarts the application, all changes would be lost, if you have more than one running process of the application the data differs, etc.  Let’s address that and add a database as a backing service in the cloud. We can do that without changes in the code. We have to create a service instance, bind it to the application and restage the applicaiton so it starts using the database. Follow the steps below.

1. List the services available for your user with ```cf marketplace```
You should see a list of services, short description and information for the service plans.
2. In our example we will use a PostgreSQL. To list more information about concrete service, use `cf marketplace -s SERVICE`:

  ```
  cf marketplace -s postgresql
  ```

3. Before your application can use a service, you need to create a service instance. Create a PostgreSQL service instance with `cf create-service SERVICE PLAN SERVICE_INSTANCE`:

  ```
  cf create-service postgresql v9.6-dev postgres
  ```

4. You should now see your newly created postgresql service. Type in the command prompt:

  ```
  cf services
  ```
You should see in the output the list of services within your space.

5. The last step is to bind the service instance to your application. You can do it with command in CF CLI `cf bind-service APP_NAME SERVICE_INSTANCE`:
```
 cf bind-service product-list postgres
```
6. Now that we already have the backing service to ensure that our application environment gets updated we re-stage our application so that the app is wired to use PostgreSQL for persistence. Use the following command `cf restage APP_NAME`:

  ```
  cf restage product-list
  ```

If you list now again the services with ```cf services```, you will see that the PostgreSQL instance is bund to the product list application.

You can again request the application once it's restaged though there will be no difference now that it uses PostgreSQL.

:bulb: **Note** An alternative to bind services to your application is via the application manifest. When you use manifest.yml to bind services you have to push the application again so that the environment gets updated and the app can use the service. E.g. for this sample you have to add this snippet and then cf push the application again:
```Config
services:
 - SERVICE_INSTANCE_NAME
```

Usually at this point you will have to consume the service instance and adapt your application code. The information how to access the service instance is available now in your application environment as an environment variable VCAP_SERVICES. In our case the SpringBoot framework provides a mechanism to detect this wired PostgreSQL service and start working with it, so that's why no need to change the application code.
