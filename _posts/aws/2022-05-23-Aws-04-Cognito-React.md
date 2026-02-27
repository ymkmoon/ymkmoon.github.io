---
title:  "[AWS] React 에서 Cognito OAuth2 사용하기"
excerpt: "React 에서 Cognito 사용하여 OAuth2 로그인과 이메일 인증을 통한 회원가입, 로그인을 해보자"

categories:
  - AWS
  
tags:
  - [AWS, Cognito, OAuth2, React]

toc: true
toc_sticky: true
toc_label: "[AWS Cognito #3]"
 
date: 2022-05-25
last_modified_at: 2022-05-25
---

- AWS Cognito 활용하기
  1.  [AWS Cognito 사용자 풀 생성하기](https://ymkmoon.github.io/Aws-02-Cognito/)
  2.  [AWS Cognito OAuth2(Google) 설정하기](https://ymkmoon.github.io/Aws-03-Cognito-OAuth2/)
  3.  [React 에서 Cognito 사용하기](https://ymkmoon.github.io/Aws-04-Cognito-React/)
  4.  [SpringBoot(Restful API) 에서 Cognito 로 토큰 검증하여 인증받기](https://ymkmoon.github.io/Aws-05-Springboot-Cognito/)

<hr/>

- 참고사이트
  - [AWS Document](https://aws.amazon.com/ko/getting-started/guides/deploy-webapp-amplify/module-two/){:target="_blank"}
  - [Amplify Document](https://docs.amplify.aws/lib/auth/start/q/platform/js/#configure-your-application){:target="_blank"}
  - [Amplify WORKSHOP](https://master.d3f5073vvso9t3.amplifyapp.com/lab2/contents/frontend/){:target="_blank"}
  - [css 커스텀](https://velog.io/@younge/AWS-Cognito%EC%99%80-Amplify%EB%A1%9C-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EA%B8%B0%EB%8A%A5-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-React){:target="_blank"}

<br>


### ``Amplify 라이브러리 설치``

특정 버전을 설치하는 이유는 최선 버전에서 ui 이슈가 있기 때문이다. 

```console
npm install @aws-amplify/ui-react^1.2.5 --save
npm install aws-amplify --save
npm install aws-amplify-react@^2.5.4 --save
```

<hr/>

### ``Amplify 설정 파일 생성``

🛠 src/assets/js/amplifyConfig.js

```js
const CognitoRegion = 'ap-northeast-2';
const CognitoUserPool = 'ap-northeast-2_xxxxxx';
const CognitoUserPoolClient = 'xxxxxxxx';
const CognitoDomainPrefix = 'toyseven-domain';

const AmplifyConfig = {
    Auth: {
        region: CognitoRegion,
        userPoolId: CognitoUserPool,
        userPoolWebClientId : CognitoUserPoolClient,
        oauth: {
            domain: `${CognitoDomainPrefix}.auth.${CognitoRegion}.amazoncognito.com`,
            // scope: ['openid'],
            redirectSignIn: `http://localhost:3000`,
            redirectSignOut: `http://localhost:3000`,
            responseType: `code`
        }
    }

};

export default AmplifyConfig;

```

<hr/>

### ``회원가입 페이지 생성``

🛠 src/pages/SignUp.js

```js
import React from 'react';
import Amplify, {Auth} from "aws-amplify";
import { AmplifyAuthenticator, withAuthenticator, AmplifySignOut  } from "@aws-amplify/ui-react";
import amplifyConfig from '../assets/js/amplifyConfig';

Amplify.configure(amplifyConfig);
const currentConfig = Auth.configure();

function SignUp() {
    Auth.currentSession().then(session => {
        console.log('session : '+JSON.stringify(session));
    }).catch(e => {
        console.log(e);
    });

  return (<>
    <AmplifyAuthenticator>
      <div>
        ONLY LOGGED IN USERS CAN SEE THIS
      </div>
    </AmplifyAuthenticator>
    <AmplifySignOut/>
  </>);
}
  
export default SignUp;
```

<hr/>


### ``라우트 설정``

🛠 App.js

```js
import SignUp from'./pages/SignUp';

<Route path='/sign-up' element={<SignUp />}  />
```

<hr/>


### ``OAuth2 로그인 테스트``

자세한 Response 값은 브라우저 콘솔에 남기도록 했으니 확인하면 된다.

![COGNITO](/assets/image/aws/AWS_Cognito_React_01.PNG)

![COGNITO](/assets/image/aws/AWS_Cognito_React_02.PNG)

![COGNITO](/assets/image/aws/AWS_Cognito_React_03.PNG)

<hr/>


### ``이메일 인증 테스트``

자세한 Response 값은 브라우저 콘솔에 남기도록 했으니 확인하면 된다.

#### 회원가입

![COGNITO](/assets/image/aws/AWS_Cognito_React_04.PNG)

#### 정보입력

![COGNITO](/assets/image/aws/AWS_Cognito_React_05.PNG)

#### 인증번호 확인

![COGNITO](/assets/image/aws/AWS_Cognito_React_06.PNG)

#### 인증번호 입력

![COGNITO](/assets/image/aws/AWS_Cognito_React_07.PNG)

#### 회원가입과 동시에 로그인 완료 

![COGNITO](/assets/image/aws/AWS_Cognito_React_08.PNG)

