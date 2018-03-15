# 샘플 응용 프로그램 복제(advanced branch)

## 예상 시간

:clock4: 5 분

## 목표

이 연습에서는 master branch가 아닌 advanced branch에 있는 고급 세션에 대한 원본 다운로드 방법을 배우고 로컬 Eclipse 환경에 가져옵니다.

# 연습과정 설명

## 1. 샘플 코드를 다운로드하십시오.
1. github에서 프로젝트의 루트로 이동합니다. https://github.com/SAP/cloud-cf-product-list-sample
2. **Branch: master**를 클릭하고 드롭다운 리스트에서 **advanced** branch를 선택합니다.
3. 이제 **Clone or download** 버튼을 클릭하고 zip 다운로드를 선택하십시오.
4. 압출을 ```cloud-cf-product-list-sample-advanced``` 이름으로 해제합니다.

## 2. Eclipse로 샘플 프로젝트 가져오기
1. Eclipse를 실행합니다.
2. 이제 샘플 프로젝트를 Maven 프로젝트로 Eclipse 작업 공간에 가져 오십시오.: Eclipse 메뉴에서 ```File```> ```Import...```를 선택합니다.
3. ```Import``` 화면에서, ```Maven``` > ```Existing Maven Projects``` 를 선택하고 ```Next```를 클릭합니다.
4. ```Import Maven Projects``` 팝업의 다음단계에서, ```Browse```, 클릭하고 아까 압축 풀어둔 ```cloud-cf-product-list-sample-advanced``` 프로젝트를 찾아 루트폴더를 지정한 다음 ```Finish``` 버튼을 클릭합니다.
5. 이제 Eclipse 프로젝트 탐색기에서 임포트한 프로젝트를 볼 수 있습니다.
