---
title:  "[GQL] GraphQL mutation"
excerpt: "Apollo mutation 사용하기"

tags:
  - [React, GraphQL]

toc: true
toc_sticky: true
toc_label: "[GraphQL mutation]"
 
date: 2022-05-16
last_modified_at: 2022-05-16
---

> 모든 소스코드는 [여기](https://github.com/ymkmoon/toyseven-react){:target="_blank"} 에서 확인 가능합니다.

### ``1. mutation 란?``

GraphQL 에서 일반적인 Get 요청에는 `query` 를 사용하고, 
Get 요청을 제외한 나머지 요청 Post, Put, Delete 등의 `mutation` 을 사용한다.

실제 테스트를 해보니 query 를 사용해도 `@rest` 옵션을 이용하면 원하는 Http Method (POST 등) 로 통신이 가능했기에 앞서 [Apollo 게시글 작성](https://ymkmoon.github.io/React-12-Apollo-Voc-Post-Question/){:target="_blank"} 에서는 query 를 이용했지만, GQL의 규약에 따라 mutation 으로 변경해보겠다.

<br>

### ``2. query -> mutation 수정하기``


🛠 src/pages/apollo/voc/ApolloVocQuestion.js

```js
// query submitQuestion -> mutation submitQuestion 으로 변경
const questionSubmitQuery = gql`
  mutation submitQuestion($body: PublishablePostInput!) {
    question(body: $body) 
      @rest(
          type: "Question" 
          path: "/voc" 
          method: "POST"
          bodyKey: "body"
      ) {
      id
    }
  }
`;
```


```js
// await client.query({ query: questionSubmitQuery, variables: {body: body} }) 
// -> 
// await client.mutate({ mutation: questionSubmitQuery, variables: {body: body} }) 으로 변경
const HandleQuestionSubmit = async({body}) => {
  const response = await client.mutate({ mutation: questionSubmitQuery, variables: {body: body} }).then(response => {
    console.log('response : '+JSON.stringify(response));
  }).catch((error) => {
    console.log('error : '+error);
  });
}
```

<br>

### ``3. 테스트``

![React](/assets/image/react/React_toyseven_react_15.PNG)

<br>

![React](/assets/image/react/React_toyseven_react_16.PNG)