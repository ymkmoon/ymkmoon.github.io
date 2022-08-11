---
title:  "[Azure] DevOps organizations - DevOps 생성과 Git Repository 배포하기"
excerpt: "Azure 에 대한 설명과 리소스 만들기"

tags:
  - [Azure]

toc: true
toc_sticky: true
toc_label: "[Azure]"
 
date: 2022-08-11
last_modified_at: 2022-08-11
---

### Azure DevOps organizations 란?

소스 코드 관리, 보고, 요구 사항 관리, 프로젝트 관리, 자동화된 빌드, 랩 관리, 테스트 및 출시 관리 기능을 제공하는 서비스이다. 

사실 설명을 덧붙이고 싶지만 위 내용으로 충분하다 싶다. 하지만 내 블로그니 몇자 더 적겠다. <br>

- `Sprint(스프린트 = 작은 기능 하나에 대한 [계획, 개발, 테스트, 기능 완료] 주기를 일컫는 말)`를 사용한 스케줄 관리
- `백로그(Backlog = 제품에서 요구하는 기능과 우선순위)` 를 통해 우선순위를 선정하여 개발 진행 
- `Repo` 를 이용한 형상관리
- `파이프라인(Pipelines = 프로세서에서 성능을 높이기 위해서 명령어 처리 과정으로 명령어 처리를 여러 단계로 나누어 단계별로 동시에 수행 하여 병렬화를 시키는 것)` 을 이용해 프로젝트를 자동으로 빌드하고 테스트
- 파이프라인을 통한 CI(지속적통합)/CD(지속적배포)

개발자가 코드를 수정하고 직접 빌드와 테스트, 배포까지 하던 걸 요즘엔 백엔드, 프론트엔드, 데브옵스 등등으로 역할을 찢어놓는 추세인데, Azure DevOps organizations 기능을 보면 업무 분담과 일감 관리, 히스토리를 한번에 관리 할 수 있다는 장점(?)이 있을 것 같다.

<hr/>

### organizations 생성

> [Azure Portal(https://portal.azure.com/#home)](https://portal.azure.com/#home) 로 접속하여 DevOps Organizations 을 생성해보자

![Azure](/assets/image/azure/Azure_DevOps_01.PNG)

> organizations 을 검색한다.

![Azure](/assets/image/azure/Azure_DevOps_02.PNG)

> Azure DevOps organizations 을 선택하면 아래와 같은 화면이 표시되는데 무척이나 오류화면 처럼 생겼다. 나도놀랐다. 당황하지말고 `My Azure DevOps Organizations` 문구를 선택 후 추가정보를 입력 후 넘어가자.

![Azure](/assets/image/azure/Azure_DevOps_03.PNG)

![Azure](/assets/image/azure/Azure_DevOps_04.PNG)

![Azure](/assets/image/azure/Azure_DevOps_05.PNG)

![Azure](/assets/image/azure/Azure_DevOps_06.PNG)

![Azure](/assets/image/azure/Azure_DevOps_07.PNG)

![Azure](/assets/image/azure/Azure_DevOps_08.PNG)

> 여기까지 진행이 됐다면 이제 얼마 남지 않았다. 프로젝트 이름(본인은 MyFirstDevOpsProject 로 설정했다)을 설정하자.

![Azure](/assets/image/azure/Azure_DevOps_09.PNG)

> Azure DevOps 가 정상적으로 생성 됐다.

![Azure](/assets/image/azure/Azure_DevOps_10.PNG)

<hr/>

### Git Repository Import

> 아래 화면에서 Repos 를 선택한다

![Azure](/assets/image/azure/Azure_DevOps_11.PNG)

> Import a repository 에서 Import 를 선택 하고 Clone 할 Git Repository 의 URL 을 입력 후 Import 한다.

![Azure](/assets/image/azure/Azure_DevOps_12.PNG)

![Azure](/assets/image/azure/Azure_DevOps_13.PNG)

> 정상적으로 완료되면 Clone 한 Repository 를 확인 할 수 있다.

![Azure](/assets/image/azure/Azure_DevOps_14.PNG)

<hr/>

### Pipeline 을 통한 배포

> 사이드 메뉴에서 Pipelines 를 선택하고 생성하기를 진행하면 아래와 같은 화면이 나온다. 화면에서 Azure Repose Git 을 선택한다.

![Azure](/assets/image/azure/Azure_DevOps_15.PNG)

> DevOps 프로젝트를 선택한다.

![Azure](/assets/image/azure/Azure_DevOps_16.PNG)

> Gradle 을 선택한다.

![Azure](/assets/image/azure/Azure_DevOps_17.PNG)

> Save and run 을 선택 후 진행하면 azure-pipelines.yml 파일이 추가 될 때의 브랜치 선택과 커밋메세지를 설정할 수 있다. 설정 완료 후 Save and run 을 다시 한번 선택하자.

![Azure](/assets/image/azure/Azure_DevOps_18.PNG)

![Azure](/assets/image/azure/Azure_DevOps_19.PNG)

> 어라? 빌드에 대한 오류가 발생했다. <br> 
`##[error]No hosted parallelism has been purchased or granted. To request a free parallelism grant, please fill out the following form https://aka.ms/azpipelines-parallelism-request` <br>
무료 병렬 처리 보조금을 요청해야 한다고 한다. 위 주소에 접속해서 양식을 작성해서 보내면 보통 2~3일 정도 걸리니 기다려보자


