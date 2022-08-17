---
title:  "[Azure] 내가 생성한 리소스, App 의 Client id, secret, tenant 찾기"
excerpt: "생성하거나 생성되어 있는 리소스와 App의 정보 찾기"

tags:
  - [Azure]

toc: true
toc_sticky: true
toc_label: "[Azure]"
 
date: 2022-08-16
last_modified_at: 2022-08-16
---

### Client id 찾기

> [Azure Portal(https://portal.azure.com/#home)](https://portal.azure.com/#home){:target="_blank"} 에 접속한다.

![Azure](/assets/image/azure/Azure_Find_Information_01.PNG)

> 왼쪽 메뉴바(삼선모양) 을 선택 후 Azure Active Directory 선택를 선택한다.

![Azure](/assets/image/azure/Azure_Find_Information_02.PNG)

> 사이드메뉴에서 관리 - 엔터프라이즈 애플리케이션 으로 접속한다.

![Azure](/assets/image/azure/Azure_Find_Information_03.PNG)

> 애플리케이션 유형을 `모든 애플리케이션` 으로 변경한다.

![Azure](/assets/image/azure/Azure_Find_Information_04.PNG)

> 아래와 같은 목록이 표시된다. 내가 찾고자 하는 서비스(or AppName) 을 선택한다.

![Azure](/assets/image/azure/Azure_Find_Information_05.PNG)

> 애플리케이션 ID(client id)를 확인 할 수 있다. 

![Azure](/assets/image/azure/Azure_Find_Information_06.PNG)

<hr/>

### Client secret 찾기

secret 은 기존 값에 대해 접근이 불가능하다. 즉 새로 생성해야 된다. <br>
그럼 새로 생성하는 방법에 대해 알아보자

> [Azure Portal(https://portal.azure.com/#home)](https://portal.azure.com/#home){:target="_blank"} 에 접속한다.

![Azure](/assets/image/azure/Azure_Find_Information_01.PNG)

> 왼쪽 메뉴바(삼선모양) 을 선택 후 Azure Active Directory 선택를 선택한다.

![Azure](/assets/image/azure/Azure_Find_Information_02.PNG)

> 사이드 메뉴에서 앱 등록을 선택한다.

![Azure](/assets/image/azure/Azure_Find_Information_07.PNG)

> secret 을 생성할 애플리케이션을 선택한다.

![Azure](/assets/image/azure/Azure_Find_Information_08.PNG)

> 인증서 및 암호를 선택한다.

![Azure](/assets/image/azure/Azure_Find_Information_09.PNG)

> 새 클라이언트 암호를 선택 후 암호를 생성하면 완료된다.

![Azure](/assets/image/azure/Azure_Find_Information_10.PNG)

![Azure](/assets/image/azure/Azure_Find_Information_11.PNG)

<hr/>

### tenant id 찾기

> [Azure Portal(https://portal.azure.com/#home)](https://portal.azure.com/#home){:target="_blank"} 에 접속한다.

![Azure](/assets/image/azure/Azure_Find_Information_01.PNG)

> 왼쪽 메뉴바(삼선모양) 을 선택 후 Azure Active Directory 선택를 선택한다.

![Azure](/assets/image/azure/Azure_Find_Information_02.PNG)

> 사이드메뉴에서 관리 - 속성 을 선택한다.

![Azure](/assets/image/azure/Azure_Find_Information_12.PNG)

> 테넌트 id 값 확인이 가능하다

![Azure](/assets/image/azure/Azure_Find_Information_13.PNG)

<hr/>
