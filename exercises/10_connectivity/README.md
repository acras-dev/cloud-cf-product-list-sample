# 연결

## 예상 시간

:clock4: 20 분

## 목표

이 과정에서는 Cloud Connector를 구성하는 방법, 클라우드 응용 프로그램에서 사용할 수있는 사내 시스템을 노출하는 방법 및 Cloud Connector를 SAP Cloud Platform의 하위 계정에 연결하는 방법에 대해 학습합니다. 클라우드 커넥터 측면의 모든 구성이 완료되면 클라우드 응용 프로그램을 확장하여 클라우드 커넥터가 제공하는 온-프레미스 시스템에 액세스하고 연결 서비스를 사용하는 방법을 배우게됩니다.

샘플 응용 프로그램의 컨텍스트에서 제품의 아이콘을 검색하는 방법을 변경합니다. 클라우드 애플리케이션의 로컬 리소스로 사용하지 않고 인터넷을 통해 액세스 할 수없는 외부 서버에 배치합니다. 연결 서비스 및 Cloud Connector를 사용하여 이 서버에 액세스하고 제품 아이콘을 가져옵니다.

# 연습과정 설명

대부분의 경우 클라우드 응용 프로그램은 온-프레미스 시스템에 액세스해야합니다. 대부분의 경우 이러한 응용 프로그램은 보안상의 이유로 인터넷에 노출되지 않습니다. 이러한 시나리오의 경우 Cloud Connector를 사용하여 일부 사내 시스템을 일부 클라우드 응용 프로그램에 표시 할 수 있습니다. 클라우드 커넥터가 구성되고 연결된 클라우드 응용 프로그램이 연결 서비스를 사용하여 액세스 할 수 있습니다.

## 단계 개요

* 로컬 시스템에서 간단한 백엔드 시스템을 시작하십시오.
* 평가판 하위 계정에 Cloud Connector를 시작하고 연결하십시오.
* 클라우드 커넥터를 구성하고 첫 번째 단계에서 백엔드 시스템을 공개하십시오.
* 연결 서비스를 사용하도록 응용 프로그램을 수정하십시오.
* 이를 업데이트하고 연결 서비스 인스턴스에 바인딩하십시오.
* 응용 프로그램을 테스트하십시오. 클라우드 커넥터에서 사용하지 않는 경우 온-프레미스 시스템에 액세스 할 수 없도록 하십시오.


# 세부 단계

## Python 세팅

온-프레미스 시스템으로서 우리는 Python의 내장 된 http 서버를 통해 노출 된 파일 시스템의 일부를 사용할 것입니다. 이를 위해 Python을 다운로드하여 설치해야합니다.
* [공식 홈페이지](https://www.python.org/ftp/python/3.6.2/python-3.6.2.exe)에서 Python을 다운로드
* 다운로드 한 실행 파일을 실행하십시오.
* 설치 마법사에서 **Install Now**를 선택하십시오.
* 설치가 완료 될 때까지 기다리십시오.

## 백엔드 시스템 준비

간단한 로컬 http 서버는 다음 단계를 사용하여 시작할 수 있습니다.
* 예제 파일 (src/main/resources/static/images)이 들어있는 폴더를 로컬 파일 시스템의 임의의 폴더에 복사합니다. 아무 파일이나 사용할 수 있습니다.
* HTTP server 시작:
	* Cloud Foundry 명령에 사용하는 CLI와는 별도의 CLI를 하나더 엽니다.
	* */images*(첫 번째 단계에서 폴더를 복사 한 위치)의 상위 폴더로 이동합니다.
	* 다음 명령을 실행하십시오.
		```
			C:\Users\student\AppData\Local\Programs\Python\Python36-32\python.exe -m http.server 10080
		```
	:warning: 서버를 시작하는 데 사용되는 CLI를 닫지 마십시오.
* 브라우저에서 http://localhost:10080 을 열어 시작한 서버가 작동하는지 확인하십시오.

	![Open Backend From Browser](/img/connectivity_backend.png?raw=true)


## 클라우드 커넥터 시작 및 연결

클라우드 커넥터를 로컬 시스템에 설치합니다. 아래 경로를 참조합니다.
https://www.sap.com/developer/tutorials/hcp-cloud-connector-setup.html

* [Cloud Connector configuration UI](https://scc.fair.sap.corp:8443) 엽니다. ID/PW: (**Administrator** : **welcome**) 
* [SAP Cloud Platform cockpit](https://account.hana.ondemand.com/cockpit) 열고 Home -> Cloud Foundry Environment -> US East (VA) Cloud Foundry 지역으로 이동하십시오.
* 평가판 하위 계정으로 이동하여 오른쪽 하단의 "refresh" 아이콘을 클릭하십시오. ID를 클립 보드에 복사하십시오.

	![Get Subaccount ID](/img/connectivity_subaccountid.png?raw=true)
* 클라우드 커넥터 UI로 이동하여 "Add Subaccount" 버튼을 클릭하십시오.

	![Add Subaccount](/img/connectivity_addsubaccount.png?raw=true)
* Region Host는 cf.us10.hana.ondemand.com이며 Subaccount Name은 위에서 클립보드에 복사한 ID입니다. 평가판 하위 계정 및 비밀번호를 만드는 데 사용 된 이메일 주소를 입력합니다. description과 display name은 당신이 편한데로 입력합니다.

	![Subaccount Information](/img/connectivity_subaccountinfo.png?raw=true)
* 모든 구성이 성공적으로 완료되면 클라우드 커넥터가 "Connected status" 메세지를 보여줍니다.

## 클라우드 커넥터 구성 및 테스트 백엔드 시스템 노출

기본적으로 백엔드 시스템은 클라우드 애플리케이션에서 접근할 수 없으므로 클라우드 커넥터에서 구성해야합니다.
* 클라우드 커넥터 UI에서, "Cloud To On-Premise" -> "Access Control" 이동해 add mapping 클릭

	![Add Mapping](/img/connectivity_addaccess.png?raw=true)
* backend type "Non SAP System" -> Protocol "HTTP" -> Internal host "localhost", Internal port "10080" -> Virtual Host & port 는 변경안해도됨 -> Principal propagation type "None" -> Finish;
* */images* 폴더와 모든 컨텐츠를 노출하십시오.
	* virtual mapping에 new resource를 추가합시오.

		![Add Resource](/img/connectivity_resourceButton.png?raw=true)
	* URL path에 **/images** 입력하고 **Path and all sub-paths** 선택합니다.

		![Resource data](/img/connectivity_addresource.png?raw=true)

이제 Cloud Connector를 연결하고 구성했으며 하나의 백엔드 시스템을 클라우드 응용 프로그램에 공개했습니다.

## 클라우드 애플리케이션 수정

이제 연결 서비스를 사용하도록 애플리케이션을 수정합니다.
* *pom.xml* 파일에 다음 종속성을 추가 하십시오
```xml
	<dependency>
	      <groupId>org.cloudfoundry.identity</groupId>
	      <artifactId>cloudfoundry-identity-client-lib</artifactId>
	      <version>3.16.0</version>
	      <scope>compile</scope>
	</dependency>
	<dependency>
              <groupId>commons-io</groupId>
              <artifactId>commons-io</artifactId>
              <version>2.5</version>
        </dependency>
```
* Eclipse에서 *com.sap.cp.cf.demoapps* 패키지에 새 클래스를 추가하고 **ConnectivityConsumer**로 이름을 지정합니다 .
* 새로 생성 된 **ConnectivityConsumer.java** 파일 내용을 다음으로 변경합니다.
```java
	package com.sap.cp.cf.demoapps;

	import java.io.IOException;
	import java.io.InputStream;
	import java.net.HttpURLConnection;
	import java.net.InetSocketAddress;
	import java.net.Proxy;
	import java.net.URI;
	import java.net.URL;

	import org.apache.commons.io.IOUtils;
	import org.cloudfoundry.identity.client.UaaContext;
	import org.cloudfoundry.identity.client.UaaContextFactory;
	import org.cloudfoundry.identity.client.token.GrantType;
	import org.cloudfoundry.identity.client.token.TokenRequest;
	import org.cloudfoundry.identity.uaa.oauth.token.CompositeAccessToken;
	import org.json.JSONArray;
	import org.json.JSONException;
	import org.json.JSONObject;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.security.core.Authentication;
	import org.springframework.security.core.context.SecurityContextHolder;
	import org.springframework.security.oauth2.provider.authentication.OAuth2AuthenticationDetails;

	import com.sap.xs2.security.container.UserInfoException;

	public class ConnectivityConsumer {
		private static final Logger logger = LoggerFactory.getLogger(Application.class);

		public byte[] getImageFromBackend(String fileName) throws IOException {
			HttpURLConnection urlConnection = null;
			URL url = null;
			try {
				// Build the URL to the requested file.
				url = new URL("http://mybackend:10080/images/" + fileName);

				// Build the connectivity proxy and set up the connection.
				Proxy proxy = getProxy();
				urlConnection = (HttpURLConnection) url.openConnection(proxy);

				// Get connectivity access token and configure the proxy authorization header.
				CompositeAccessToken accessToken = getAccessToken();
				urlConnection.setRequestProperty("Proxy-Authorization", "Bearer " + accessToken);

				// Set connection timeouts.
				urlConnection.setConnectTimeout(10000);
				urlConnection.setReadTimeout(60000);

				// Get the user JWT token and configure the connectivity authentication header.
				String token = getClientOAuthToken();
				urlConnection.setRequestProperty("SAP-Connectivity-Authentication", "Bearer " + token);

				// Execute request, returning the response as a byte array.
				urlConnection.connect();
				InputStream in = urlConnection.getInputStream();
				return IOUtils.toByteArray(in);

			} catch (Exception e) {
				String messagePrefix = "Connectivity operation failed with reason: ";
				logger.error(messagePrefix, e);
			} finally {
				if (urlConnection != null) {
					urlConnection.disconnect();
				}
			}
			return null;
		}

		// Parse the credentials for a given service from the environment variables.
		private JSONObject getServiceCredentials(String serviceName) throws JSONException {
			JSONObject jsonObj = new JSONObject(System.getenv("VCAP_SERVICES"));
			JSONArray jsonArr = jsonObj.getJSONArray(serviceName);
			return jsonArr.getJSONObject(0).getJSONObject("credentials");
		}

		// Create a HTTP proxy from the connectivity service credentials.
		private Proxy getProxy() throws JSONException {
			JSONObject credentials = getServiceCredentials("connectivity");
			String proxyHost = credentials.getString("onpremise_proxy_host");
			int proxyPort = Integer.parseInt(credentials.getString("onpremise_proxy_port"));
			return new Proxy(Proxy.Type.HTTP, new InetSocketAddress(proxyHost, proxyPort));
		}

		// Get JWT token for the connectivity service from UAA
		private CompositeAccessToken getAccessToken() throws Exception {
			JSONObject connectivityCredentials = getServiceCredentials("connectivity");
			String clientId = connectivityCredentials.getString("clientid");
			String clientSecret = connectivityCredentials.getString("clientsecret");

			// Make request to UAA to retrieve JWT token
			JSONObject xsuaaCredentials = getServiceCredentials("xsuaa");
			URI xsUaaUri = new URI(xsuaaCredentials.getString("url"));

			UaaContextFactory factory = UaaContextFactory.factory(xsUaaUri).authorizePath("/oauth/authorize")
					.tokenPath("/oauth/token");

			TokenRequest tokenRequest = factory.tokenRequest();
			tokenRequest.setGrantType(GrantType.CLIENT_CREDENTIALS);
			tokenRequest.setClientId(clientId);
			tokenRequest.setClientSecret(clientSecret);

			UaaContext xsUaaContext = factory.authenticate(tokenRequest);
			return xsUaaContext.getToken();
		}

		private String getClientOAuthToken() throws Exception {
			Authentication auth = SecurityContextHolder.getContext().getAuthentication();
			if (auth == null) {
				throw new Exception("User not authenticated");
			}
			OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails) auth.getDetails();
			return details.getTokenValue();
		}
	}

```
**getImageFromBackend** 메소드는 온-프레미스 시스템에 대한 요청을 처리합니다. 연결 서비스를 통해 서버에서 이미지를 가져 오는 작업이 수행됩니다. **getServiceCredentials** 메소드는 애플리케이션 환경 변수의 JSON 포맷된 인증 정보를 판독하는 데 사용됩니다. **getProxy**의 메소드는 프록시로 연결 서비스 endpoint를 구성합니다. **getAccessToken** 메소드는 xsuaa 서비스에서 JWT 액세스 토큰을 요청합니다. 토큰은 연결 프록시의 권한 부여에 사용됩니다. **getClientOAuthToken**는 클라이언트 JWT 토큰을 검색합니다. 응용 프로그램은 **SAP-Connectivity-Authentication** 헤더를 통해 이 토큰을 전달할 책임이 있습니다. 이는 클라우드 커넥터에서 구성이 이루어진 하위 계정에 대한 터널을 열기 위해 연결 서비스에서 필요합니다.
* 이제 온-프레미스 시스템에 호출할 수있는 메소드를 추가하십시오. 이를 위해 모든 *images/\<file\>* 요청을 매핑하여 *\<file\>* 을 인수로 전달하여 *getImageFromBackend(String fileName)* 메소드를 수행하십시오. **Controller.java**의 내용을 다음과 같이 변경하십시오.

```java
	package com.sap.cp.cf.demoapps;

	import java.io.IOException;
	import java.util.Collection;

	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.http.MediaType;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.ResponseBody;
	import org.springframework.web.bind.annotation.RestController;

	@RestController
	public class Controller {

		private static final Logger logger = LoggerFactory.getLogger(Application.class);

		@Autowired
		private ProductRepo productRepo;

		@GetMapping("/productsByParam")
		public Collection<Product> getProductByName(@RequestParam(value = "name") final String name) {
			logger.info("***First - Retrieving details for '{}'.", name);
			logger.info("***Second - Retrieving details for '{}'.", name);
			return productRepo.findByName(name);
		}

		@ResponseBody
		@GetMapping(value = "/images/{imageFile:.+}", produces = MediaType.IMAGE_JPEG_VALUE)
		public byte[] getIcon(@PathVariable String imageFile) throws IOException {
			ConnectivityConsumer connConsumer = new ConnectivityConsumer();
			return connConsumer.getImageFromBackend(imageFile);
		}
	}
```		

:bulb: **Note:** **Application.java**를 보면 아이콘의 URL이 컨트롤러에 매핑 한 패턴과 일치 함을 알 수 있습니다 (예 : *"images/HT-1000.jpg"*) . 즉, 목록 항목이 로드 될 때 아이콘이 온-프레미스 시스템에서 검색됩니다.

## 연결 서비스 인스턴스 만들기 및 응용 프로그램에 바인딩
* LI에서 다음 명령을 실행하여 연결 서비스 인스턴스를 만듭니다.
```
 	cf create-service connectivity lite connInstance<Unique-ID>
```
* 응용 프로그램에 바인딩합니다.
```
	cf bind-service product-list connInstance<Unique-ID>
```

## 응용 프로그램 올리기
* *manifest.yml* 파일을 수정해 연결 인스턴스에 대한 바인딩을 포함시킵니다.
```
	services:
	    - ...
	    - connInstance<Unique-ID>
```
* Eclipse에서 project 오른쪽 클릭 -> Run As -> Maven install;
* CLI에서 application을 올립니다.
```
	cf push product-list
```

## 응용 프로그램 테스트
* 응용 프로그램을 열고 온-프레미스 시스템에서 static 파일을 가져 왔는지 확인하십시오. Python 서버를 시작하는 데 사용한 CLI를 사용하여 온-프레미스 시스템에 대한 호출을 모니터링 할 수 있습니다.

	![Console](/img/connectivity_onpremiseconsole.png?raw=true)
* 노출 된 백엔드 시스템을 사용하지 않도록 설정합니다. 클라우드 커넥터 UI ->> "Cloud To On-Premise" -> Access Control -> 생성한 가상 매핑을 선택하고 아래의 리소스 테이블에서 */images* 리소스를 비활성화합니다.

	![Disable resource](/img/connectivity_disable.png?raw=true)

	이제 클라우드 애플리케이션이 백엔드 시스템에 액세스 할 수 없도록 하십시오.  
