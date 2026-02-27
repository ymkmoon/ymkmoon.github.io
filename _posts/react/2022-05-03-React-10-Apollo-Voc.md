---
title:  "[React] 게시판 만들기 #9 : Apollo & GraphQL 을 이용한 게시판 목록"
excerpt: "React Apollo & Rest API 게시판 목록 만들기"

categories:
  - React

tags:
  - [React, Apollo, GraphQL]

toc: true
toc_sticky: true
toc_label: "[Apollo GQL 게시판 목록]"
 
date: 2022-05-03
last_modified_at: 2022-05-03
---

- 리액트로 게시판 만들기
  1.  [VSCode 와 Node 설치하기](https://ymkmoon.github.io/React-02-React/)
  2.  [리액트 프로젝트와 네비게이션 생성하기](https://ymkmoon.github.io/React-03-Navigation/)
  3.  [페이지 이동을 위한 Route 설정하기](https://ymkmoon.github.io/React-04-Route/)
  4.  [Axios 설치 및 사용해보기](https://ymkmoon.github.io/React-05-Axios/)
  5.  [Axios 를 이용하여 게시판 목록 만들기](https://ymkmoon.github.io/React-06-Voc/)
  6.  [Axios 를 이용하여 게시판 상세보기 만들기](https://ymkmoon.github.io/React-07-Voc-Question/)
  7.  [Axios 를 이용하여 게시판 작성 만들기](https://ymkmoon.github.io/React-08-Voc-Post-Question/)
  8.  [Apollo GraphQL 을 설치 및 사용해보기](https://ymkmoon.github.io/React-09-Apollo/)
  9.  [Apollo GraphQL 을 이용하여 게시판 목록 만들기](https://ymkmoon.github.io/React-10-Apollo-Voc/)
  10.  [Apollo GraphQL 을 이용하여 게시판 상세보기 만들기](https://ymkmoon.github.io/React-11-Apollo-Voc-Question/)
  11.  [Apollo GraphQL 을 이용하여 게시판 작성 만들기](https://ymkmoon.github.io/React-12-Apollo-Voc-Post-Question/)

<hr/>

> React 소스코드는 [여기](https://github.com/ymkmoon/toyseven-react){:target="_blank"} 에서 확인 가능합니다.

> 개발에 사용한 Restful API 는 [여기](https://github.com/ymkmoon/toyseven){:target="_blank"} 에서 확인 가능합니다.

<hr/>

- 참고사이트
  - [APOLLO DOCS](https://www.apollographql.com/docs/react/api/link/apollo-link-rest/){:target="_blank"}
  


### ``Route 설정``

- 작성 페이지로 이동 할 수 있도록 Route 설정과 매개변수명을 지정한다.

🛠 App.js

```js
// 추가내용
import ApolloVoc from './pages/apollo/voc/ApolloVoc';

function App() {

    return (
      <>
      <Router>
        <Navbar />
        <Routes>
          <Route exact path='/' element={<Home />} />
          <Route path='/station' element={<Station />} />
          <Route path='/voc' element={<Voc />} />
          <Route path='/voc/:vocId' element={<VocView />}  />
          <Route path='/voc/question' element={<VocQuestion />}  />
          <Route path='/apollo/station' element={<ApolloStation />}  />
          // 추가내용
          <Route path='/apollo/voc' element={<ApolloVoc />}  />
        </Routes>
      </Router>
      </>
    );
}
export default App;
```


### ``Component 생성``

🛠 pages/apollo/voc/ApolloVoc.js

```js
import React, {useState } from 'react';
import { ApolloClient, InMemoryCache, gql } from '@apollo/client';
import { RestLink } from 'apollo-link-rest';

import CommonTable from '../../../components/table/CommonTable';
import CommonTableColumn from '../../../components/table/CommonTableColumn';
import CommonTableRow from '../../../components/table/CommonTableRow';

// Set `RestLink` with your endpoint
const restLink = new RestLink({ uri: "http://127.0.0.1:8000/toyseven" });

// Setup your client
const client = new ApolloClient({
  cache: new InMemoryCache(),
  link: restLink
});

const query = gql`
  query getQuestions {
    questions 
      @rest(
          type: "Question" 
          path: "/voc" 
          method: "GET"
      ) {
      id
      title
      createdAt
      username
    }
  }
`;


function GetData() {
  const [questions, setQuestions] = useState({});
  client.query({ query }).then(response => {
    setQuestions(response.data.questions);
  });

  const item = questions && Object.values(questions).map((voc) => (
    <CommonTableRow key={voc.id}>
      <CommonTableColumn>{voc.id}</CommonTableColumn>
      <CommonTableColumn>{voc.title}</CommonTableColumn>
      <CommonTableColumn>{voc.createdAt}</CommonTableColumn>
      <CommonTableColumn>{voc.username}</CommonTableColumn>
    </CommonTableRow>
  ));

  return item;
}


function ApolloVoc() {
  const item = GetData();

  return (<>
    <CommonTable headersName={['글번호', '제목', '등록일', '작성자']}>
      {item}
    </CommonTable>
  </>);
}
  
export default ApolloVoc;
```
<br>

![React](/assets/image/react/React_toyseven_react_11.PNG)

<br>

