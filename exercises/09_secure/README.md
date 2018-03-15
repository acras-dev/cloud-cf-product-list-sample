# OAuth 2.0으로 보안된 애플리케이션 만들기

## 예상 시간

:clock4: 60 분

## 목표
이 과정에서 유연한 인증 프레임 워크인 OAuth 2.0을 사용하여 샘플 응용 프로그램을 보호하는 방법을 학습합니다. OAuth 2.0의 승인 코드 부여는 승인 된 사용자에게만 애플리케이션 및 데이터에 대한 액세스 권한을 부여하는 탁월한 보안 메커니즘을 제공합니다. SAP XS Advanced Application Router, SAP XS UAA OAuth 인증 서비스 및 Spring Boot는 역할을 구성하고 사용자에게 할당하고 마지막으로 응용 프로그램에서 역할 검사를 구현하는 탁월한 도구입니다.

# 연습과정 설명
SAP Cloud Platform에 배포 된 마이크로 서비스는 제한없이 접근할 수 있습니다. 인증된 사용자에게만 액세스를 제한하려면 [OAuth 2.0](https://tools.ietf.org/html/rfc6749) 같은 적절한 보안 메커니즘을 구현해야 합니다.

## Steps overview
AP Cloud Platform에서 OAuth 2.O를 사용하여 샘플 응용 프로그램을 보호하려면 다음 단계가 필요합니다.

* 1 단계 : Application Security Descriptor의 정의
* 2 단계 : XS UAA 서비스 생성 및 구성
* 3 단계 : 필수 보안 라이브러리 추가
* 4 단계 : Spring Security 프레임 워크의 설정
* 5 단계 : XS Advanced Application Router
* 6 단계 : Trust configuration

:warning: 아직 샘플 응용 프로그램이 준비되지 않았다면 아래 url을 참고해 Eclipse에 가져 오십시오.
```(https://github.com/SAP/cloud-cf-product-list-sample/tree/master/exercises/11_clonebranch)```

### 1 단계 : Application Security Descriptor의 정의

**이 단계는 master branch에서 작업하는 경우에만 필수입니다. advanced branch인 경우 원리만 이해하고 아무 것도 변경할 필요가 없습니다.**

Application Security Descriptor는 제품 목록 응용 프로그램에 접근하는 데 사용할 인증 방법 및 권한 유형의 세부 정보를 정의합니다. 샘플 응용 프로그램은 이 정보를 사용하여 scope 검사를 수행합니다. scope를 사용하면 세밀한 사용자 권한을 얻을 수 있습니다. Spring Security는 모든 HTTP endpoints에서 각 HTTP 메소드의 scope를 확인할 수 있게합니다. 범위는 [JSON Web Tokens (JWTs)](https://tools.ietf.org/html/rfc7519)에 의해 전달되며,이 토큰은 차례로 [XS UAA 서비스](https://help.sap.com/viewer/4505d0bdaf4948449b7f7379d24d0f0d/1.0.12/en-US/17acf1ac0cf84487a3199c51b28feafd.html)에서 발행됩니다.

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

**master branch & advanced branch 모두 필수입니다.**

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

### 3 단계 : 필수 보안 라이브러리 추가

**이 단계는 master branch에서 작업하는 경우에만 필수입니다. advanced branch인 경우 원리만 이해하고 아무 것도 변경할 필요가 없습니다.**

응용 프로그램 보안을 위해 클래스 보안에 Spring Security를 추가해야 합니다. Spring Security는 응용 프로그램에서 Spring Security를 구성함으로써 BASIC 인증을 통해 자동으로 모든 HTTP endpoints를 보호합니다. [Java Web Token (JWT)](https://tools.ietf.org/html/rfc7519) 와 함께 OAuth 2.0을 사용하기 때문에 Spring OAUTH와 Spring JWT 의존성을 추가해야 합니다.

오프라인 JWT 유효성 검사를 사용하려면 SAP XS 보안 라이브러리도 추가해야합니다. 라이브러리는 `cloud-cf-product-list-sample-advanced/libs`에 저장됩니다. 최신 버전은 [Service Marketplace](https://launchpad.support.sap.com/#/softwarecenter/template/products/%20_APP=00200682500000001943&_EVENT=DISPHIER&HEADER=Y&FUNCTIONBAR=N&EVENT=TREE&NE=NAVIGATE&ENR=73555000100200004333&V=MAINT&TA=ACTUAL&PAGE=SEARCH/XS%20JAVA%201)에서 다운로드 할 수 있습니다. 작성 당시 최신 버전은 `XS_JAVA_4-70001362`입니다.

**Note:** 최신 버전의 SAP XS 보안 라이브러리를 사용하는 경우 에는 `pom.xml` 안에 버전 번호를 적용해야합니다.

* `cloud-cf-product-list-sample-advanced/libs/XS_JAVA_4-70001362.ZIP` 압축풀기
* 다음을 실행하여 SAP XS 보안 라이브러리를 로컬 Maven 저장소에 설치하십시오.

```shell
cd cloud-cf-product-list-sample-advanced/libs
mvn clean install
```
* advanced `pom.xml` 파일에는 이미 다음 종속성이 추가되어 있습니다.

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

**Note:** master branch로 시작한 경우 단위 테스트가 실패합니다. 단위 테스트에 대한 인증을 사용하지 않으려면 `ControllerTest` 클래스를 향상시켜야합니다.

* `ControllerTests` 클래스에 `@AutoConfigureMockMvc(secure = false)` 추가
* Eclipse에서 프로젝트 빌드 (`Context Menu -> Run As -> Maven install`) -> Result: BUILD SUCCESS
* Spring Boot App으로 프로젝트를 실행하십시오. (`Context Menu -> Run As -> Spring Boot App`)
* 당신의 브라우저에 `localhost:8080` 호출 -> 인증이 필요하다는 창이 표시됩니다.

모든 HTTP endpoints가 보호되고 샘플 응용 프로그램에 접근할 수 없습니다. 다시 액세스하려면 스프링 보안을 구성해야합니다.

### 4 단계 : Spring Security 프레임 워크의 설정

**이 단계는 master branch에서 작업하는 경우에만 필수입니다. advanced branch인 경우 원리만 이해하고 아무 것도 변경할 필요가 없습니다.**

* advanced branch 에서는 클래스 `com.sap.cp.cf.demoapps.ConfigSecurity.java`가 아래 범위 검사 및 오프라인 토큰 유효성 검사가 포함되 만들어졌습니다.

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

이제 모든 종단점이 상태 종단점을 제외하고 차단됩니다. 아래 과정으로 확인 할 수 있습니다.
* `Maven Install` 실행
* `product-list` 오른쪽클릭하고 `Run As -> Spring Boot App`
* http://localhost:8080/health 접근

### 5 단계 : XS Advanced Application Router

**master branch & advanced branch 모두 필수입니다. advanced branch 경우 'npm install'단계만 실행해야 합니다. 동작원리를 좀더 이해하고자 여전히 살펴볼 필요가 있습니다.**

[XS Advanced Application Router](https://github.com/acras-dev/cloud-cf-product-list-sample/tree/advanced/src/main/approuter/README.md)는 여러 가지 응용 프로그램 (microservices)로 구성된 비즈니스 애플리케이션에 대한 single entry point을 제공하는 데 사용됩니다. 백엔드 마이크로 서비스에 대한 요청을 디스패치하고 역방향 프록시 역할을 합니다. 어떤 요청을 _destinations_ 로 전달해야 하는지를 결정하는 규칙을 _routes_ 라고 합니다. application router는 사용자를 인증하고 사용자 정보를 전파하도록 구성 할 수 있으며 static content를 제공 할 수 있습니다.

**Note** application router가 어떤 식 으로든 백엔드 microservices을 숨기지 않습니다. application router를 거치지 않고도 직접 접근할 수 있습니다. 따라서 백엔드 마이크로 서비스는 JWT 토큰의 유효성을 검사하고 적절한 범위 검사를 구현하여 모든 엔드 포인트를 보호 해야 합니다.

* `product-list/src/main/approuter`에 application router를 다운로드하여 설치하십시오.
  * 새폴더 만들기: `mkdir product-list/src/main/approuter`
  * 경로 변경: `cd product-list/src/main/approuter`
  * 새 파일을 만들어 application router의 버전을 지정: `vi package.json`

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

* :bulb: 오직 advanced branch 경우 `cloud-cf-product-list-sample-advanced/src/main/approuter`에서 `npm install` 실행

```shell
    approuter$ npm config set @sap:registry https://npm.sap.com
    approuter$ npm install @sap/approuter
```

* `vi src/main/approuter/xs-app.json`에 대상 및 경로를 정의하여 application router를 구성합니다.

```json
{
  "routes": [{
    "source": "^/",
    "target": "/",
    "destination": "products-destination"
  }]
}
```

* `manifest.yml`에 application router 추가하기

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

* `cf push` 명령어로 approuter를 추가한 응용프로그램 [올리기](../04_push)

### 6 단계 : Trust configuration

**master branch & advanced branch 모두 필수입니다.**

이제 업무용 사용자 또는 최종 사용자를 위해 응용 프로그램에 접근하는 방법을 알아 보겠습니다.
- 브라우저에서 `approuter` 응용 프로그램을 실행하고 사용자 자격 증명으로 로그온하십시오
- Insufficient scope for this resource 라는 오류를 보게됩니다.
<br><br>
![Authorizations](/img/security_cockpit_0.png?raw=true)
<br><br>
최종 사용자에게 필요한 인증을 할당해야합니다. 응용 프로그램의 권한은 security.json을 사용하여 xsuaa 권한 서비스에 등록됩니다. 조종석에서 응용 프로그램에 대한 이러한 권한을 볼 수 있습니다.
- `Org --> Space --> Applications --> approuter` 이동합니다. [this is the front end application]
- `Security` 그룹을 확장하고 `Roles` UI로 이동합니다.
<br><br>
![Authorizations](/img/security_cockpit_1.png?raw=true)
<br><br>
- UI는 응용 프로그램에서 정의한 역할을 나열합니다.
<br><br>
![Authorizations](/img/security_cockpit_2.png?raw=true)
<br><br>
- 최종 사용자에 대한 액세스를 제공하려면 위의 역할을 최종 사용자에게 할당해야합니다. 역할을 최종 사용자에게 직접 할당 할 수는 없으므로 RoleCollection을 만들고 필요한 역할을 RoleCollection에 추가해야합니다.
- `Subaccount --> Security --> RoleCollections` 이동합니다. [expand the security group to see this entry]
<br><br>
![Authorizations](/img/security_cockpit_3.png?raw=true)
<br><br>
- **New Role Collection** 버튼을 클릭하십시오.
<br><br>
![Authorizations](/img/security_cockpit_4.png?raw=true)
<br><br>
- RoleCollection의 이름과 설명을 입력하고 **Save** 버튼을 클릭하십시오.
<br><br>
![Authorizations](/img/security_cockpit_5.png?raw=true)
<br><br>
- RoleCollection으로 이동하고 **Add Role** 버튼을 클릭하십시오.
<br><br>
![Authorizations](/img/security_cockpit_6.png?raw=true)
<br><br>
- 팝업 대화 상자에서 샘플 애플리케이션, 역할 템플리트 및 역할을 선택하십시오. **Save** 버튼을 클릭하십시오.
<br><br>
![Authorizations](/img/security_cockpit_7.png?raw=true)
<br><br>
- 다음 단계로 이 RoleCollection을 사용자에게 할당하십시오.  `Subaccount --> Trust Configuration` 이동합니다. [expand the security group to see this entry]
- 기본 trust configuration인 **SAP ID Service**를 클릭하십시오.
<br><br>
![Authorizations](/img/security_cockpit_8.png?raw=true)
<br><br>
- 최종 사용자에게 RoleCollection을 할당하기 전에 사용자는 최소한 한 번 SAP ID 서비스에 로그온해야합니다.
- 로그온 URL은 https://$identityzone.$uaaDomain 입니다. 이것은 xsuaa 바인딩 자격 증명에서 식별 할 수 있습니다. (`cf env approuter` & `xsuaa.credentials.url`에서 확인)
- 이제 'Role Collection Assignment'UI에서 현재 계정에 로그온하는 데 사용 된 사용자 ID를 입력하고 **Show Assignments** 버튼을 클릭한다.
- 사용자에게 현재 역할 컬렉션 할당을 나열하고 사용자에게 새 역할 컬렉션을 추가 할 수도 있습니다.
- **Add Assignment** 버튼을 클릭하십시오.
<br><br>
![Authorizations](/img/security_cockpit_9.png?raw=true)
<br><br>
- 팝업 대화 상자에서 최근 정의한 역할 컬렉션을 선택하고 **Add Assignment** 버튼을 클릭하십시오.
<br><br>
![Authorizations](/img/security_cockpit_10.png?raw=true)
<br><br>
- 이제 사용자는 애플리케이션에 접근할 수 있어야합니다.
- 브라우저에서 응용 프로그램을 실행하고 자격 증명으로 로그인하십시오. 제품 목록을 볼 수 있어야합니다.
