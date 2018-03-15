# OAuth 2.0으로 보안된 애플리케이션 만들기

## 예상 시간

:clock4: 60 분

## 목표
이 과정에서 유연한 인증 프레임 워크인 OAuth 2.0을 사용하여 샘플 응용 프로그램을 보호하는 방법을 학습합니다. OAuth 2.0의 승인 코드 부여는 승인 된 사용자에게만 애플리케이션 및 데이터에 대한 액세스 권한을 부여하는 탁월한 보안 메커니즘을 제공합니다. SAP XS Advanced Application Router, SAP XS UAA OAuth 인증 서비스 및 Spring Boot는 역할을 구성하고 사용자에게 할당하고 마지막으로 응용 프로그램에서 역할 검사를 구현하는 탁월한 도구입니다.

# 연습과정 설명
SAP Cloud Platform에 배포 된 마이크로 서비스는 제한없이 접근할 수 있습니다. 인증된 사용자에게만 액세스를 제한하려면 [OAuth 2.0](https://tools.ietf.org/html/rfc6749) 같은 적절한 보안 메커니즘을 구현해야 합니다.

## Steps overview
AP Cloud Platform에서 OAuth 2.O를 사용하여 샘플 응용 프로그램을 보호하려면 다음 단계가 필요합니다.

* 1 단계 : 응용 프로그램 보안 설명자 정의
* 2 단계 : XS UAA 서비스 생성 및 구성
* 3 단계 : 필수 보안 라이브러리 추가
* 4 단계 : Spring Security 프레임 워크의 설정
* 5 단계 : XS 고급 응용 프로그램 라우터 추가
* 6 단계 : 신뢰 구성

:warning: 아직 샘플 응용 프로그램이 준비되지 않았다면 아래 url을 참고해 Eclipse에 가져 오십시오.
```(https://github.com/SAP/cloud-cf-product-list-sample/tree/master/exercises/11_clonebranch)```

### 1 단계 : 응용 프로그램 보안 설명자 정의

**이 단계는 master branch에서 작업하는 경우에만 필수입니다. advanced branch인 경우 동작원리만 이해하면 되고 아무 것도 변경할 필요가 없습니다.**

응용 프로그램 보안 설명자는 샘플 응용 프로그램에 액세스하는 데 사용할 인증 방법 및 권한 유형의 세부 정보를 정의합니다. 샘플 응용 프로그램은 이 정보를 사용하여 범위 검사를 수행합니다. 범위를 사용하면 세밀한 사용자 권한을 얻을 수 있습니다. Spring Security는 모든 HTTP 엔드포인트에서 각 HTTP 메소드의 범위를 확인할 수 있습니다. 범위는 [JSON Web Tokens (JWTs)](https://tools.ietf.org/html/rfc7519) which in turn are issued by the [XS UAA Service](https://help.sap.com/viewer/4505d0bdaf4948449b7f7379d24d0f0d/1.0.12/en-US/17acf1ac0cf84487a3199c51b28feafd.html)에 의해 전달되며, 이 토큰은 차례로 XS UAA 서비스에서 발행됩니다.

* `src/main/security/` 안에 `xs-security.json` 파일을 생성합니다.
* 아래 JSON content를 복사/붙여넣기 합니다.

```json
{
	"xsappname": "product-list",
	"tenant-mode": "dedicated",
	"scopes":
	[
		{
			"name": "$XSAPPNAME.read",
			"description": "With this scope, all endpoints of the product-list app can be read."
		}
	],

	"role-templates":
	[
		{
			"name": "UserRoleTemplate",
			"description": "Role to call the product-list service",
			"scope-references":
			[
				"$XSAPPNAME.read"
			]
		}
	]
}
```

### 2 단계 : XS UAA 서비스 생성 및 구성

**master branch & advanced branch 모두 필수**

사용자에게 샘플 응용 프로그램에 대한 액세스 권한을 부여하려면 이 응용 프로그램에 대한 XS UAA 서비스 인스턴스를 만들어야 합니다. XSUAA 서비스 인스턴스는 바운드 애플리케이션에 대한 OAuth 2.0 클라이언트 역할을 합니다.
* CF CLI에 사용할 Cloud Foundry를 알려줘야 합니다. 이렇게 하려면 Cloud Foundry 시험판을 만든 Cloud Foundry 지역의 Cloud Controller에 API endpoint를 설정해야 합니다.
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
:bulb: **Note:** [SAP Cloud Platform Documentation](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/350356d1dc314d3199dca15bd2ab9b0e.html) 문서에서 Cloud Foundry Environment를 사용할 수 있는 여러 지역에 대한 API endpoint를 찾을 수 있습니다.

* 사용자 계정으로 로그인하십시오. 명령 프롬프트에서 다음을 입력하십시오.
	```
	cf login
	```

	SAP Cloud Platform 평가판 계정에 등록 할 때 사용한 전자 메일 및 암호를 입력하라는 메시지가 나타납니다.

	```
	Email> enter your e-mail
	Password> password for your user
	```
* marketplace 조회합니다. `cf m`
* XS UAA 서비스 인스턴스 생성합니다. `cf create-service xsuaa application xsuaa -c ./src/main/security/xs-security.json`
* 서비스에 XS UAA 서비스 인스턴스를 추가합니다. `manifest.yml`: `- xsuaa`

### Step 3: Adding required security libraries

**This step is mandatory only if you work on the master branch. For the advanced branch you can go through it to understand what is happening (no need to change anything)**

To secure the application we have to add Spring Security to the classpath. By configuring Spring Security in the application, Spring Boot automatically secures all HTTP endpoints with BASIC authentication. Since we want to use OAuth 2.0 together with [Java Web Tokens (JWT)](https://tools.ietf.org/html/rfc7519) instead, we need to add the Spring OAUTH and Spring JWT dependencies as well.

To enable offline JWT validation the SAP XS Security Libraries need to be added as well. The libraries are stored in `cloud-cf-product-list-sample-advanced/libs`. The latest version can be downloaded from the [Service Marketplace](https://launchpad.support.sap.com/#/softwarecenter/template/products/%20_APP=00200682500000001943&_EVENT=DISPHIER&HEADER=Y&FUNCTIONBAR=N&EVENT=TREE&NE=NAVIGATE&ENR=73555000100200004333&V=MAINT&TA=ACTUAL&PAGE=SEARCH/XS%20JAVA%201). At the time of writing the latest version is `XS_JAVA_4-70001362`.

**Note:** Be aware to adapt the version number in your `pom.xml` in case you are using a newer version of the SAP XS Security Libraries.

* Unzip `cloud-cf-product-list-sample-advanced/libs/XS_JAVA_4-70001362.ZIP`
* Install SAP XS Security Libraries to your local maven repo by executing:

```shell
cd cloud-cf-product-list-sample-advanced/libs
mvn clean install
```
* The following dependencies are already added to the advanced `pom.xml` file:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-jwt</artifactId>
</dependency>
<dependency>
    <groupId>com.sap.xs2.security</groupId>
    <artifactId>security-commons</artifactId>
    <version>0.22.2</version>
</dependency>
<dependency>
    <groupId>com.sap.xs2.security</groupId>
    <artifactId>java-container-security</artifactId>
    <version>0.22.2</version>
</dependency>
<dependency>
    <groupId>com.unboundid.components</groupId>
    <artifactId>json</artifactId>
    <version>1.0.0</version>
</dependency>
<dependency>
    <groupId>com.sap.security.nw.sso.linuxx86_64.opt</groupId>
    <artifactId>sapjwt.linuxx86_64</artifactId>
    <version>1.0.0</version>
</dependency>
<dependency>
    <groupId>com.sap.security.nw.sso.ntamd64.opt</groupId>
    <artifactId>sapjwt.ntamd64</artifactId>
    <version>1.0.0</version>
</dependency>
<dependency>
    <groupId>com.sap.security.nw.sso.linuxppc64.opt</groupId>
    <artifactId>sapjwt.linuxppc64</artifactId>
    <version>1.0.0</version>
</dependency>
<dependency>
    <groupId>com.sap.security.nw.sso.darwinintel64.opt</groupId>
    <artifactId>sapjwt.darwinintel64</artifactId>
    <version>1.0.0</version>
</dependency>
```

**Note:** In case you started with the master branch the unit tests will fail. To disable authentication for the unit tests we need to enhance the `ControllerTest` class.

* Add `@AutoConfigureMockMvc(secure = false)` to `ControllerTests` class
* Build the project in Eclipse (`Context Menu -> Run As -> Maven install`) -> Result: BUILD SUCCESS
* Run the project as Spring Boot App (`Context Menu -> Run As -> Spring Boot App`)
* Call `localhost:8080` from your browser -> a window is popping up informing us that authentication is required

All HTTP endpoints are secured and the Product List application is inaccessible. To regain access, we need to configure the Spring Security.

### Step 4: Configuration of the Spring Security framework

**This step is mandatory only if you work on the master branch. For the advanced branch you can go through it to understand what is happening (no need to change anything)**

* In the advanced branch, a new class `com.sap.cp.cf.demoapps.ConfigSecurity.java` was created including the following scope checks and offline token validations.

```java
package com.sap.cp.cf.demoapps;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;

import static org.springframework.http.HttpMethod.GET;

@Configuration
@EnableWebSecurity
@EnableResourceServer
public class ConfigSecurity extends ResourceServerConfigurerAdapter {

  @Value("${vcap.services.xsuaa.credentials.xsappname:product-list}")
	private String xsAppName;

	// configure Spring Security, demand authentication and specific scopes
	@Override
	public void configure(final HttpSecurity http) throws Exception {

		// @formatter:off
    http.sessionManagement()
				.sessionCreationPolicy(SessionCreationPolicy.NEVER).and()
				.authorizeRequests()
				.antMatchers(GET, "/health").permitAll()
				.antMatchers(GET, "/**").access(String.format("#oauth2.hasScope('%s.%s')", xsAppName, "read"))
				.anyRequest().denyAll(); // deny anything not configured above
		// @formatter:on
	}

  // offline token validation
	@Bean
	protected static SAPOfflineTokenServicesCloud offlineTokenServicesBean() {
		return new SAPOfflineTokenServicesCloud();
	}
}
```

Now all endpoints are blocked except the health endpoint. You can verify that by:
* running `Maven Install`
* right clicking on `product-list` and then `Run As -> Spring Boot App`
* clicking on the following link http://localhost:8080/health

### Step 5: Adding the XS Advanced Application Router

**This step is mandatory for both the master and the advanced branch. For the latter, only the 'npm install' step has to be executed. You should still go entirely through it to have a better understanding of what is happening though.**

The [XS Advanced Application Router](https://github.com/SAP/cloud-cf-product-list-sample/blob/advanced/src/main/approuter/README.md) is used to provide a single entry point to a business application that consists of several different apps (microservices). It dispatches requests to backend microservices and acts as a reverse proxy. The rules that determine which request should be forwarded to which _destinations_ are called _routes_. The application router can be configured to authenticate the users and propagate the user information. Finally, the application router can serve static content.

**Note** that the application router does not hide the backend microservices in any way. They are still directly accessible bypassing the application router. So, the backend microservices _must_ protect all their endpoints by validating the JWT token and implementing proper scope checks.

* Download and install the application router in `product-list/src/main/approuter`
  * Create a new folder: `mkdir product-list/src/main/approuter`
  * Change directory to: `cd product-list/src/main/approuter`
  * Create a new file to specify the version of the application router: `vi package.json`

```json
{
    "name": "approuter",
    "dependencies": {
        "@sap/approuter": "^2.9.1"
    },
    "scripts": {
        "start": "node node_modules/@sap/approuter/approuter.js"
    }
}
```

*  Run `npm install`
:bulb: For the advanced branch only this command has to be executed from `cloud-cf-product-list-sample-advanced/src/main/approuter`

```shell
    approuter$ npm config set @sap:registry https://npm.sap.com
    approuter$ npm install @sap/approuter
```

* Configure the application router by defining the destinations and routes: `vi src/main/approuter/xs-app.json`

```json
{
  "routes": [{
    "source": "^/",
    "target": "/",
    "destination": "products-destination"
  }]
}
```

* Add the application router to the `manifest.yml`

```yml
---
applications:
# Application
- name: product-list
  instances: 1
  memory: 896M
  host: product-list-YOUR_BIRTH_DATE
  path: target/product-list.jar
  buildpack: https://github.com/cloudfoundry/java-buildpack.git#v4.3
  services:
    - postgres
    - xsuaa
  env:
    SAP_JWT_TRUST_ACL: '[{"clientid" : "*", "identityzone" : "*"}]'
# Application Router
- name: approuter
  host: approuter-YOUR_BIRTH_DATE
  path: src/main/approuter
  buildpack: nodejs_buildpack
  memory: 128M
  env:
    destinations: >
      [
        {"name":"products-destination",
         "url":"https://product-list-YOUR_BIRTH_DATE.cfapps.eu10.hana.ondemand.com",
         "forwardAuthToken": true}
      ]
  services:
    - xsuaa
```

* [Push the product list application togehter with a approuter](https://github.com/SAP/cloud-cf-product-list-sample/tree/master/exercises/04_push) to your cloud foundry space: `cf push`

### Step 6: Trust configuration

**This step is mandatory for both master and advanced branch**

Now let us see how to enable access to the application for the business users or end-users.
- Launch the `approuter` application in the browser and logon with your user credentials
- You will get an error, Insufficient scope for this resource
<br><br>
![Authorizations](/img/security_cockpit_0.png?raw=true)
<br><br>
In order to enable access, the end-users should be assigned the required authorizations. The authorizations of the application is registered with the authorization services, xsuaa, using the security.json. You can view these authorizations for the application in the Cockpit.
- Navigate to the `Org --> Space --> Applications --> approuter` [this is the front end application]
- Expand the `Security` group and navigate to `Roles` UI
<br><br>
![Authorizations](/img/security_cockpit_1.png?raw=true)
<br><br>
- The UI lists the roles defined by the application
<br><br>
![Authorizations](/img/security_cockpit_2.png?raw=true)
<br><br>
- In order to provide access to the end-users, the above role has to be assigned to the end-user. Roles can't be directly assigned to the end-users, you will have to create RoleCollection and add the required Roles to the RoleCollection.
- Navigate to the `Subaccount --> Security --> RoleCollections` [expand the security group to see this entry]
<br><br>
![Authorizations](/img/security_cockpit_3.png?raw=true)
<br><br>
- Click on button **New Role Collection**
<br><br>
![Authorizations](/img/security_cockpit_4.png?raw=true)
<br><br>
- Enter the name and description for the RoleCollection and click on button **Save**
<br><br>
![Authorizations](/img/security_cockpit_5.png?raw=true)
<br><br>
- Navigate into the RoleCollection and click on button **Add Role**
<br><br>
![Authorizations](/img/security_cockpit_6.png?raw=true)
<br><br>
- In the pop-up dialog, choose the sample application, the role template and the role. Click on button **Save**
<br><br>
![Authorizations](/img/security_cockpit_7.png?raw=true)
<br><br>
- As a next step, assign this RoleCollection to the user. Navigate to the `Subaccount --> Trust Configuration` [expand the security group to see this entry]
- Click on the link **SAP ID Service** - the default trust configuration
<br><br>
![Authorizations](/img/security_cockpit_8.png?raw=true)
<br><br>
- Before assigning the RoleCollection to the end-user, the user should have logged on to SAP ID Service at least once
- The logon URL is https://$identityzone.$uaaDomain. This can be identified from the xsuaa binding credentials (`cf env approuter` and look for `xsuaa.credentials.url`)
- Now, in the `Role Collection Assignment' UI, enter your user id used to logon to the current account and click on button **Show Assignments**
- It lists the current Role Collection assignment to the user and also allows to add new Role Collections to the user
- Click on button **Add Assignment**
<br><br>
![Authorizations](/img/security_cockpit_9.png?raw=true)
<br><br>
- In the pop-up dialog, choose the Role Collection you have defined recently and click on button **Add Assignment**
<br><br>
![Authorizations](/img/security_cockpit_10.png?raw=true)
<br><br>
- Now, the user should be able to access the application
- Launch the application on the browser and login with your credentials. You should be able to see the product list
