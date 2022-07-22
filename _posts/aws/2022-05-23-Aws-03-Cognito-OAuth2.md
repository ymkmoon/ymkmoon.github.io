---
title:  "[AWS] Cognito 와 Google OAuth2 연동하기"
excerpt: "Aws Cognito 와 Google OAuth2 연동하기"

tags:
  - [AWS, Cognito, OAuth2]

toc: true
toc_sticky: true
toc_label: "[AWS Cognito #2]"
 
date: 2022-05-23
last_modified_at: 2022-05-23
---

- 참고사이트
  - [AWS Document](https://aws.amazon.com/ko/premiumsupport/knowledge-center/cognito-google-social-identity-provider/){:target="_blank"}
  - [AWS Document](https://aws.amazon.com/ko/blogs/mobile/understanding-amazon-cognito-user-pool-oauth-2-0-grants/){:target="_blank"}


### ``OAuth2 동의화면``

#### 프로젝트 생성
구글에 접속하여 개발자 콘솔을 이용해 프로젝트를 생성한다.

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_01.PNG)


#### 사용지 인증 정보

`API 및 서비스` -> `사용자 인증 정보` 탭으로 이동한다.

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_02.PNG)

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_03.PNG)

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_04.PNG)


#### OAuth 동의화면

동의화면이 설정 안되어 있을 경우 자동으로 동의화면 생성으로 넘어갑니다.

- 애플리케이션 이름 : 구글 로그인 시 사용자에게 노출될 애플리케이션 이름
- 사용자 지원 이메일 : 사용자 동의 화면에서 노출될 이메일 주소 
  보통은 서비스의 help 이메일 주소를 사용하지만, 여기서는 본인의 이메일 주소를 입력하면 된다.

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_05.PNG)

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_06.PNG)

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_07.PNG)


#### 범위

변경하지 않고 다음 단계로 넘어간다. <br>

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_08.PNG)


#### 테스트 사용자

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_09.PNG)



<br> 


### ``사용자 인증정보``


#### 이름

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_03.PNG)

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_11.PNG)

#### 승인된 자바스크립트 원본

cognito console 에서 도에인 부분을 입력합니다.<br>
(https://toyseven-domain.auth.ap-northeast-2.amazoncognito.com)

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_10.PNG)

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_12.PNG)


#### 승인된 리다이렉션 URI

congnito console 의 도메인 뒤에 Endpoint(/oauth2/idpresponse) 를 추가합니다.<br>
 - https://toyseven-domain.auth.ap-northeast-2.amazoncognito.com/oauth2/idpresponse

 [AWS Document](https://aws.amazon.com/ko/blogs/mobile/understanding-amazon-cognito-user-pool-oauth-2-0-grants/){:target="_blank"}

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_13.PNG)


#### 생성

클라이언트 ID 와 Pssword 는 Cognito OAuth2 를 등록할 때 필요하기 떄문에 따로 저장해둡니다.

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_14.PNG)


<br>


### ``Congito OAuth2 설정``

#### 자격 증명 공급자

Cognito 콘솔에서 `자격 증명 공급자` 탭을 선택합니다. <br>
구글을 선택하고 위에서 확인한 클아이언트 ID 와 Password 를 입력 합니다.

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_15.PNG)


#### 앱 클라이언트 확인

`앱 통합` -> `앱 클라이언트 설정` 에서 활성화 된 자격 증명 공급자에 `Google` 이 추가 된 걸 확인 할 수 있고, 체크 후 저장한다.

![AWS](/assets/image/aws/AWS_Cognito_OAuth2_16.PNG)



### ``4. 마치며 ``

여기까지 하면 Cognito 와 OAuth2 연동이 완료 됐고, 다음 챕터에서는 React 에서 로그인이 가능하도록 설정해보겠습니다.