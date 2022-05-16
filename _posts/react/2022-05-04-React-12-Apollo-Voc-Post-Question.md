---
title:  "[React] Apollo GQL #4 게시글 작성하기"
excerpt: "React Apollo & Rest API 게시글 작성하기"

tags:
  - [React, Apollo, GQL]

toc: true
toc_sticky: true
toc_label: "[React Apollo GQL #4]"
 
date: 2022-05-03
last_modified_at: 2022-05-03
---

### ``1. Voc List에 링크 추가``

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

### ``2. Route 설정``

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


### ``3. 게시글 작성 페이지 생성``

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
          path: "/voc" 
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
  const response = await client.query({ query: questionSubmitQuery, variables: {body: body} }).then(response => {
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
  const [username, setUsername] = useState('unknown');
  const [stationId, setStationId] = useState('ST-4');

  const body = {
      categoryId : categoryId,
      title: title,
      content: content,
      username: username,
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

![VMWare](/assets/image/react/React_toyseven_react_13.PNG)

<br>

![VMWare](/assets/image/react/React_toyseven_react_14.PNG)