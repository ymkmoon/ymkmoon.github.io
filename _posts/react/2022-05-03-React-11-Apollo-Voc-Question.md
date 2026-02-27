---
title:  "[React] 게시판 만들기 #10 : Apollo & GraphQL 을 이용한게시판 상세보기"
excerpt: "React Apollo & Rest API 게시글 상세보기"

categories:
  - React

tags:
  - [React, Apollo, GraphQL]

toc: true
toc_sticky: true
toc_label: "[Apollo GQL 게시판 상세보기]"
 
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

### ``Voc List에 링크 추가``

- 목록 페이지에서 Link 를 생성해준다. (이동 하면서 매개변수를 가지고 이동)

🛠 pages/apollo/voc/Voc.js

```js
const item = (Object.values(data)).map((voc) => (
  <CommonTableRow key={voc.id}>
    <CommonTableColumn>{voc.id}</CommonTableColumn>
    <CommonTableColumn>
      // 추가내용
      <Link to={`/apollo/voc/${voc.id}`}>
        {voc.title}
      </Link>
    </CommonTableColumn>
    <CommonTableColumn>{voc.createAt}</CommonTableColumn>
    <CommonTableColumn>{voc.username}</CommonTableColumn>
  </CommonTableRow>
));
```


### ``Route 설정``

- 작성 페이지로 이동 할 수 있도록 Route 설정과 매개변수명을 지정한다.

🛠 App.js

```js
// 추가 내용
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
          <Route path='/apollo/voc' element={<ApolloVoc />}  />
          // 추가내용 : 매개변수명을 vocId 로 지정
          <Route path='/apollo/voc/:vocId' element={<ApolloVocView />}  />
        </Routes>
      </Router>
      </>
    );
}
export default App;
```


### ``상세보기 페이지 생성``


- 상세보기 페이지에서는 매개변수를 이용해 Api Request

🛠 pages/apollo/voc/VocView.js

```js
import React, { useState } from 'react';
import { ApolloClient, InMemoryCache, gql } from '@apollo/client';
import { RestLink } from 'apollo-link-rest';
import { useParams } from 'react-router-dom';

import './ApolloVocView.css';

const restLink = new RestLink({ uri: "http://127.0.0.1:8000/toyseven" });
const client = new ApolloClient({
  cache: new InMemoryCache(),
  link: restLink
});

const query = gql`
  query getVoc($vocId: Int!) { // 매개변수로 받은 vocId 의 타입 검증
    voc(vocId: $vocId) // vocId 라는 변수에 매개변수로 받은 값($vocId)을 저장한다.
      @rest(
          type: "Voc" 
          path: "/voc/search/{args.vocId}" // 아규먼트 중에서 값을 꺼낸다.
          method: "GET"
      ) {
          question {
            id
            title
            createdAt
            content
          }
          answer {
            id
            adminName
            createdAt
            content
          }
    }
  }
`;

  
function GetData(vocId) {
  const [question, setQuestion] = useState({});
  const [answer, setAnswer] = useState({});

  // variables: {vocId: vocId} 을 통해 매개변수를 query 로 넘겨준다.
  client.query({ query, variables: {vocId: vocId} }).then(response => {
    setQuestion(response.data.voc.question);
    setAnswer(response.data.voc.answer);
  });

  const item =  (<>
    <h2 align="center">게시글 상세정보</h2>
    <div className="voc-view-wrapper">
        <div className="voc-view-row">
            <label>게시글 번호</label>
            <label>{ question.id }</label>
        </div>
        <div className="voc-view-row">
            <label>제목</label>
            <label>{ question.title }</label>
        </div>
        <div className="voc-view-row">
            <label>작성일</label>
            <label>{ question.createdAt }</label>
        </div>
        <div className="voc-view-row">
            <label>내용</label>
            <div>
                {
                question.content
                }
            </div>
        </div>
    </div></>)

    return item;
}

function VocView() {
  const{vocId} = useParams();
  const item = GetData(vocId);

  return (<>
    <div>
        {item}
    </div>
  </>);
}
  
export default VocView;
```

<br>

🛠 pages/apollo/voc/ApolloVocView.css

```css
.voc-view-wrapper {
    width: 60%;
    margin: 0 auto;
}

.voc-view-go-list-btn {
    border: 0;
    padding: 10px;
    background-color: #ffd9d9;
}

.voc-view-row {
    margin: 10px 0;
    display: flex;
}

.voc-view-row>label:first-child {
    margin: 10px 0;
    width: 30%;
    font-weight: bold;
}

.voc-view-row>*:nth-child(2) {
    margin: 10px 0;
    width: 70%;
}
```

![React](/assets/image/react/React_toyseven_react_12.PNG)