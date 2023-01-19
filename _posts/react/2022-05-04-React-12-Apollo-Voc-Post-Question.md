---
title:  "[React] 게시판 만들기 #11 : Apollo & GraphQL 을 이용한 게시글 작성"
excerpt: "React Apollo & Rest API 게시글 작성하기"

tags:
  - [React, Apollo, GraphQL]

toc: true
toc_sticky: true
toc_label: "[Apollo GQL 게시판 작성]"
 
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

- 목록 페이지에서 Link 를 생성해준다.

🛠 components/apollo/voc/ApolloVocHeader.css

```css
.voc-header {
    width: 80%;
    margin: 0 auto;
    /* text-align: center; */
    border-spacing: 0;
    display: block;
  }
```

🛠 components/apollo/voc/ApolloVocHeader.js

```js
import React from 'react';
import { Link } from 'react-router-dom';
import './ApolloVocHeader.css';

const ApolloVocHeader = props => {
  const { headersName, children } = props;

  return (
    <div className="voc-header">
        <h2 align="left">Q/A</h2>
        <Link to='/apollo/voc/question'>
            <button align="right" className="voc-view-go-list-btn" >
            게시글 작성
            </button>
        </Link>
    </div>
  )
}

export default ApolloVocHeader;
```

🛠 pages/apollo/voc/ApolloVoc.js

```js
// 추가내용
import ApolloVocHeader from '../../../components/apollo/voc/ApolloVocHeader';


function ApolloVoc() {
  const item = GetData();

  return (<>
    // 추가내용
    <VocHeader></VocHeader>
    <CommonTable headersName={['글번호', '제목', '등록일', '작성자']}>
      {item}
    </CommonTable>
  </>);
}
  
export default ApolloVoc;
```

### ``Route 설정``

- 작성 페이지로 이동 할 수 있도록 Route 설정과 매개변수명을 지정한다.

🛠 App.js

```js
// 추가내용
import ApolloVocQuestion from './pages/apollo/voc/ApolloVocQuestion';

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
          <Route path='/apollo/voc/:vocId' element={<ApolloVocView />}  />
          // 추가내용
          <Route path='/apollo/voc/question' element={<ApolloVocQuestion />}  />
        </Routes>
      </Router>
      </>
    );
}
export default App;
```


### ``게시글 작성 페이지 생성``

🛠 src/pages/apollo/voc/ApolloVocQuestion.js

```js
import React, { useEffect, useState } from 'react';
import { ApolloClient, InMemoryCache, gql } from '@apollo/client';
import { RestLink } from 'apollo-link-rest';

import './ApolloVocView.css';

const restLink = new RestLink({ uri: "http://127.0.0.1:8000/toyseven" });
const client = new ApolloClient({
  cache: new InMemoryCache(),
  link: restLink
});

const categoryQuery = gql`
  query getCategories {
    categories 
      @rest(
          type: "Category" 
          path: "/voc/category" 
          method: "GET"
      ) {
      id
      name
      displayName
    }
  }
`;

const questionSubmitQuery = gql`
  query submitQuestion($body: PublishablePostInput!) {
    question(body: $body) 
      @rest(
          type: "Question" 
          path: "/voc/question" 
          method: "POST"
          bodyKey: "body"
      ) {
      id
    }
  }
`;

function GetCategory() {
  const [category, setCategory] = useState({});
  useEffect(() => {
    client.query({ query: categoryQuery }).then(response => {
      setCategory(response.data.categories);
    }).catch((error) => {
      console.log(error);
    });
  }, []);

  const categories = (Object.values(category)).map((item) => (
    <option key={item.id} value={item.id}>
      {item.displayName}
    </option>
  ));

  return categories;
}

const HandleQuestionSubmit = async({body}) => {

  const headers = {
    'Content-Type' : 'application/json',
    'Authorization' : "Bearer cognito 의 access token"
  }
  
  const response = await client.mutate({ mutation: questionSubmitQuery, variables: {body: body}, context: {headers: headers} }).then(response => {
    console.log('response : '+JSON.stringify(response));
  }).catch((error) => {
    console.log('error : '+error);
  });
}

function ApolloVocQuestion() {
  const categories = GetCategory();

  const [categoryId, setCategoryId] = useState(1);
  const [title, setTitle] = useState('');
  const [email, setEmail] = useState('');
  const [content, setContent] = useState('');
  const [stationId, setStationId] = useState('ST-4');

  const body = {
      categoryId : categoryId,
      title: title,
      content: content,
      stationId: stationId,
      email: email
    }  

  return (<>
    <h2 align="center">게시글 작성</h2>
    <div className="voc-view-wrapper">
        <div className="voc-view-row">
            <label>문의 유형</label>
            <select onChange={(event) => setCategoryId(event.target.value)}>
              {categories}
            </select>
        </div>
        <div className="voc-view-row">
            <label>email</label>
            <input onChange={(event) => setEmail(event.target.value)}></input>
        </div>
        <div className="voc-view-row">
            <label>제목</label>
            <input onChange={(event) => setTitle(event.target.value)}></input>
        </div>
        <div className="voc-view-row">
            <label>내용</label>
            <textarea onChange={(event) => setContent(event.target.value)}></textarea>
        </div>
    <button className="voc-view-go-list-btn" onClick={() => HandleQuestionSubmit({body})}>등록</button>
    </div>
    </>);
}
  
export default ApolloVocQuestion;
```

<br>

![React](/assets/image/react/React_toyseven_react_13.PNG)

<br>

![React](/assets/image/react/React_toyseven_react_14.PNG)