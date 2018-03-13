# 연습문제 06: Blue-Green 배포

## 예상 시간

:clock4: 10 분

## 연습문제 설명

1. `manifest.yml` 파일을 열고 응용 프로그램 이름을 product-list에서 product-list-green로 변경하십시오. 호스트도 다음 `product-list-YOUR_BIRTH_DATE-green`으로 변경하십시오.

2. `cf push`를 사용해 응용 프로그램을 사용하여 올리기

3. 모든 들어오는 요청이 새 응용 프로그램 버전 (green)으로 이동하도록 라우터를 전환하십시오. 명령을 사용하여 녹색 애플리케이션에 대한 원래 URL 경로를 입력하십시오.
    ```
    cf map-route GREEN_APP_NAME DOMAIN -n BLUE_APP_HOSTNAME
    ```
    
    지역에 따라 `DOMAIN` 값은 아래를 참조해 변경:
    - `cfapps.us10.hana.ondemand.com` for Las Vegas
    - `cfapps.eu10.hana.ondemand.com` for Barcelona
    

    **결과:** `cf map-route` 명령 후 Cloud Foundry 라우터는 임시 URL에 대한 트래픽을 Green 애플리케이션으로 계속 전송합니다. 몇 초 내에 Cloud Foundry 라우터는 Blue 및 Green 버전의 애플리케이션간에 원래의 생산 URL에 대한 트래픽의로드 밸런싱을 시작합니다.


4. Green 버전이 예상대로 실행되는지 확인한 후 Blue 버전으로의 경로를 매핑 해제 하고 **cf unmap-route** 명령을 사용하여 Blue 버전으로의 라우팅 요청을 중지 합니다.
    ```
    cf unmap-route BLUE_APP_NAME DOMAIN -n HOSTNAME
    ```
    **결과:** Cloud Foundry 라우터가 Blue 버전으로의 트래픽 전송을 중지합니다. 이제는 생산적인 도메인의 모든 트래픽이 Green 버전으로 전송됩니다.
