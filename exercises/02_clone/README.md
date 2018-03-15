# 샘플 응용 프로그램 복제

## 예상 시간

:clock4: 5 분

## 목표

이 연습에서는 Product List 이라는 샘플 응용 프로그램의 원본을 복제하고 로컬 Eclipse 환경에서 가져 와서 빌드하는 방법을 학습합니다. 처음부터 시작하는 기본 실습 세션의 일부로 실제 개발할 Product List 응용 프로그램의 원본 코드입니다. 기본 실습 세션에서 참조 용으로 사용하거나 일부 스니펫이나 파일을 쉽게 복사 할 수 있습니다.

# 연습과정 설명

## 1 zip으로 샘플 코드 다운로드
1. github에서 프로젝트의 루트로 이동합니다. https://github.com/SAP/cloud-cf-product-list-sample
2. **Clone or download** 버튼을 눌러 다운로드
<br><br>
![Download ZIP](/img/github_download_zip.png?raw=true)
<br><br>

3. 원하는 디렉토리에 다운로드 파일의 압축을 풉니다. cloud-cf-product-list-sample 라는 새로운 디렉토리가 생성됩니다.

## 2. Eclipse로 샘플 프로젝트 가져 오기
1. Eclipse를 실행합니다.
2. 이제 샘플 프로젝트를 Maven 프로젝트로 Eclipse 작업 공간에 가져 오십시오.: Eclipse 메뉴에서 ```File```> ```Import...```를 선택합니다.
3. ```Import``` 화면에서, ```Maven``` > ```Existing Maven Projects``` 를 선택하고 ```Next```를 클릭합니다.
<br><br>
![Import Maven Project](/img/import_maven_project.png?raw=true)
<br><br>
4. ```Import Maven Projects``` 팝업의 다음단계에서, ```Browse```, 클릭하고 아까 압축 풀어둔 ```cloud-cf-product-list-sample-master``` 프로젝트를 찾아 루트폴더를 지정한 다음 ```Finish``` 버튼을 클릭합니다.
5. 이제 Eclipse에서 프로젝트를 가져옵니다. 아래 스크린 샷과 같이 프로젝트 탐색기에서 프로젝트를 볼 수 있습니다.  
<br><br>
![Import Maven Project](/img/imported_project_eclipse.png?raw=true)
<br><br>

## 3. Maven을 사용하여 Eclipse에서 프로젝트를 빌드하십시오.  

1. 이제 Eclipse에서 프로젝트를 빌드하십시오. 프로젝트 탐색기에서 프로젝트를 선택하고 컨텍스트 메뉴를 연뒤 ```Run As``` > ```Maven build...``` 를 클릭합니다.
<br><br>
![Import Maven Project](/img/run_maven_build.png?raw=true)
<br><br>

2. ```Edit Configuration``` 팝업이 나오면 ```Goals``` 란에 ```clean install``` 입력하고 ```Run```을 클릭합니다. Maven 빌드는 오류없이 완료되어야 합니다.
<br><br>
![Import Maven Project](/img/maven_clean_install.png?raw=true)
<br><br>
