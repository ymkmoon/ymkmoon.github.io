---
title:  "[Azure] Key Vault 생성 & Spring Boot 에서 사용하기"
excerpt: "CLI 환경에서 Azure Key Vault 생성 및 스프링부트에서 사용하기"

tags:
  - [Azure]

toc: true
toc_sticky: true
toc_label: "[Azure]"
 
date: 2022-08-12
last_modified_at: 2022-08-12
---

- 참고사이트
  - [Azure Docs](https://docs.microsoft.com/ko-kr/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-azure-key-vault){:target="_blank"}


### Azure Key Valut 란?

보안 비밀 저장소를 제공하는 클라우드 서비스이며 키, 암호, 인증서 및 기타 비밀을 안전하게 저장할 수 있다.

Key Vault를 사용하면 개발자가 코드에 보안 정보를 저장할 필요가 없으며, Key Vault는 애플리케이션 비밀의 스토리지를 중앙 집중화하여 비밀이 유출될 가능성을 크게 줄인다. 또한 Key Vault를 사용하면 HSM(하드웨어 보안 모듈) 지원 키와 비밀을 안전하게 저장할 수 있다.

<hr/>

### 키 자격 증명 모음 만들기

> [Azure Portal(https://portal.azure.com/#home)](https://portal.azure.com/#home){:target="_blank"} 에서 `Azure Cloud Shell`을 연다.

![Azure](/assets/image/azure/Azure_Key_Vault_01.PNG)

> 브라우저 하단에 Shell 이 열리며 Bash 와 PowerShell 중 뭘 선택할지에 대해 묻는데, 난 Bash 를 선택했다.

![Azure](/assets/image/azure/Azure_Key_Vault_02.PNG)

> 스토리지를 만들고 진행하면 CLI 환경이 열린다.

![Azure](/assets/image/azure/Azure_Key_Vault_03.PNG)

![Azure](/assets/image/azure/Azure_Key_Vault_04.PNG)

<hr/>

#### Azure 로그인

> CLI 에서 Azure 계정에 대한 접근을 위해 로그인을 진행하자.

```console
az login
```

위 명령어를 입력하면 url 과 코드값을 주는데 이 페이지로 접속 후 코드를 입력해 로그인을 진행하면 된다.

![Azure](/assets/image/azure/Azure_Key_Vault_05.PNG)

![Azure](/assets/image/azure/Azure_Key_Vault_06.PNG)

![Azure](/assets/image/azure/Azure_Key_Vault_07.PNG)

![Azure](/assets/image/azure/Azure_Key_Vault_08.PNG)

> 로그인이 완료되면 이미지와 같이 내 정보(구독)가 출력된다. (빨간색 네모로 표시 된 id 값은 아래에서 guid 지정 시 사용)

![Azure](/assets/image/azure/Azure_Key_Vault_09.PNG)

```console
az account list
```

위 명령어를 통해서도 내 구독정보에 대해 나열 할 수 있다.

![Azure](/assets/image/azure/Azure_Key_Vault_10.PNG)

<hr/>

#### guid 지정

> Azure에 사용하려는 구독에 대한 GUID를 지정(구독에서 id 에 해당하는 값을 입력하여 guid 를 지정한다.)

```console
az account set -s $YOUR_ID
```
![Azure](/assets/image/azure/Azure_Key_Vault_11.PNG)

<hr/>

#### 서비스 주체 만들기

> 아래 명령어를 입력하고 나오는 출력값은 인증정보가 포함되어 절대 노출시켜서는 안된다. (아래 명령어 실행 시 필요한 값이니 메모장에 따로 저장해두자. app id 값은 포탈에서 찾을 수 있지만, secret 은 기존값에 대해 접근이 불가능해 분실하게 되면 재생성 해야한다.)

```console
az ad sp create-for-rbac --name $원하는이름 --role Contributor --scopes /subscriptions/Azure구독ID
az ad sp create-for-rbac --name testapp --role Contributor --scopes /subscriptions/Azure구독ID
```

![Azure](/assets/image/azure/Azure_Key_Vault_12.PNG)

<hr/>

### 리소스 그룹 생성

Key Vault 및 App Service 를 보관할 리소스 그룹을 만들기를 진행한다.

#### 지역 리스트

> 어떤 지역을 선택 할 지 지역 리스트를 확인해보자. (한국은 koreacentral)

```console
az account list-locations --output table
```

![Azure](/assets/image/azure/Azure_Key_Vault_13.PNG)

<hr/>

#### 리소스 그룹 생성

> Key Vault 및 App Service 앱을 보관할 리소스 그룹 생성

```console
az group create --name testapp(서비스주체의 이름) --location koreacentral
```

![Azure](/assets/image/azure/Azure_Key_Vault_14.PNG)

<hr/>

#### 리소스 그룹에 Key Vault 생성

> 아래 명령어를 입력 후 생성에 성공하면 결과값으로 url 이 출력된다.

```console
az keyvault create \
    --resource-group testapp(리소스그룹명) \
    --name testappkv20220810(키볼트이름설정) \
    --enabled-for-deployment true \
    --enabled-for-disk-encryption true \
    --enabled-for-template-deployment true \
    --location koreacentral(로케이션) \
    --query properties.vaultUri \
    --sku standard
```

|매개변수|설명|
|------|---|
|enabled-for-deployment|Key Vault 배포 옵션을 지정|
|enabled-for-disk-encryption|Key Vault 암호화 옵션을 지정|
|enabled-for-template-deployment|Key Vault 암호화 옵션을 지정|
|location|리소스 그룹을 호스팅할 Azure 지역을 지정|
|name|Key Vault의 고유한 이름을 지정|
|query|응답에서 Key Vault URI를 검색|
|sku|Key Vault SKU 옵션을 지정|

![Azure](/assets/image/azure/Azure_Key_Vault_15.PNG)

<hr/>

#### Key Vault 구성

> 관리 ID의 취득 및 목록 작업을 허용 하도록 한다.

```console
az keyvault set-policy --name testappkv20220810(키볼트이름) --spn 서비스주체의AppId --secret-permissions get list
```

|매개변수|설명|
|------|---|
|name|Key Vault의 이름|
|spn|위의 az ad sp create-for-rbac 명령 출력의 appId|
|secret-permissions|명명된 보안 주체에서 허용할 작업 목록|

![Azure](/assets/image/azure/Azure_Key_Vault_16.PNG)

![Azure](/assets/image/azure/Azure_Key_Vault_17.PNG)

<hr/>

#### Key Vault Secret 생성

```console
az keyvault secret set --name "aaaa" --vault-name "testAppKeyVault20220810(키볼트이름)" --value "1234"
```

|매개변수|설명|
|------|---|
|name|Secret 이름을 지정|
|value|Secret Value를 지정|
|value-name|앞의 Key Vault 이름을 지정|

![Azure](/assets/image/azure/Azure_Key_Vault_18.PNG)

<hr/>

### Spring Boot 에서 사용하기

#### dependency 추가

> build.gradle 파일에 아래 내용을 추가한다.

```gradle
// https://mvnrepository.com/artifact/com.azure/azure-core
implementation 'com.azure:azure-core:1.17.0'
// https://mvnrepository.com/artifact/com.azure.spring/azure-spring-boot-starter-keyvault-secrets
implementation 'com.azure.spring:azure-spring-boot-starter-keyvault-secrets:3.5.0'
```

<hr/>

#### Key Vault 정보 추가

> application.yml 파일에 azure 에 대한 정보를 추가한다.

```yml
azure:
  keyvault:
    client-id: # 서비스주체의 appId
    client-key: # 서비스주체의 password
    enabled: true
    tenant-id: # 서비스주체의 tenant
    uri: https://ymkappkv.vault.azure.net/
```

#### 사용하기

Java 파일에서 @Value 어노테이션을 이용하여 Azure 에서 내가 원하는 key 에 대한 value 를 받아올 수 있다.

```java
import org.springframework.beans.factory.annotation.Value;

// aaaa 라는 key 에 저장 된 value 를 가지고온다.
@Value("${aaaa}")
public String keyAaaa;
```

