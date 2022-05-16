---
title:  "[React] Apollo GQL #2 게시판 목록"
excerpt: "React Apollo & Rest API 게시판 목록 만들기"

tags:
  - [React, Apollo, GQL]

toc: true
toc_sticky: true
toc_label: "[React Apollo GQL #2]"
 
date: 2022-05-03
last_modified_at: 2022-05-03
---

- 참고사이트
  - <https://www.apollographql.com/docs/react/api/link/apollo-link-rest/>{:target="_blank"}


### ``1. Route 설정``

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


### ``2. Component 생성``

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

![VMWare](/assets/image/react/React_toyseven_react_11.PNG)

<br>

