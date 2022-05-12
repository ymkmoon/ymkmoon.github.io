---
title:  "[React] Axios #3 게시글 작성"
excerpt: "React 게시글 입력하기"

tags:
  - [React, Axios]

toc: true
toc_sticky: true
toc_label: "[React Axios #3]"
 
date: 2022-04-29
last_modified_at: 2022-04-29
---

- 참고 블로그
  - https://antdev.tistory.com/80

### ``1. Voc List에 링크 추가``

- 목록 페이지에서 Link 를 생성해준다.

🛠 components/voc/VocHeader.css

```css
.voc-header {
    width: 80%;
    margin: 0 auto;
    /* text-align: center; */
    border-spacing: 0;
    display: block;
  }
```

🛠 components/voc/VocHeader.js

```js
import React from 'react';
import { Link } from 'react-router-dom';
import './VocHeader.css';

const VocHeader = props => {
  const { headersName, children } = props;

  return (
    <div className="voc-header">
        <h2 align="left">Q/A</h2>
        <Link to='/voc/question'>
            <button align="right" className="voc-view-go-list-btn" >
            게시글 작성
            </button>
        </Link>
    </div>
  )
}

export default VocHeader;
```

🛠 pages/voc/Voc.js

```js
// 추가내용
import VocHeader from '../../components/voc/VocHeader'; 


function Voc() {
  const item = GetData();

  return (<>
    // 추가내용
    <VocHeader></VocHeader>
    <CommonTable headersName={['글번호', '제목', '등록일', '작성자']}>
      {item}
    </CommonTable>
  </>);
}
  
export default Voc;
```

### ``2. Route 설정``

- 작성 페이지로 이동 할 수 있도록 Route 설정과 매개변수명을 지정한다.

🛠 App.js

```js
// 추가내용
import VocQuestion from './pages/voc/VocQuestion';

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
          // 추가내용
          <Route path='/voc/question' element={<VocQuestion />}  />
        </Routes>
      </Router>
      </>
    );
}
export default App;
```


### ``3. 작성 페이지 생성``

🛠 src/pages/voc/VocQuestion.css

```css
.voc-question-wrapper {
    width: 80%;
    margin: 0 auto;
}
```




🛠 src/pages/voc/VocQuestion.js

```js
import React, { useEffect, useState } from 'react';
import axios from 'axios';

import './VocView.css';

function GetCategory() {
  const [category, setCategory] = useState({});

  useEffect(() => {
    axios.get('http://127.0.0.1:8000/toyseven/voc/category').then((response)=> {
      setCategory(response.data);
    })
  }, []);

  const categories = (Object.values(category)).map((item) => (
    <option key={item.id} value={item.id}>
      {item.displayName}
    </option>
  ));

  return categories;
}

const HandleQuestionSubmit = async({body}) => {
  const response = await axios.post('http://127.0.0.1:8000/toyseven/voc', body).then((response) => {
    console.log('status : '+response.status);
  }).catch((error) => {
    console.log('error : '+error);
  });
}

function VocQuestion() {
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
      email: email,
      stationId: stationId
    }

  return (<>
    <h2 align="center">게시글 작성</h2>
    <div className="voc-view-wrapper">
        <div className="voc-view-row">
            <label>문의 유형</label>
            <select onChange={(event) => setCategoryId(parseInt(event.target.value))}>
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
  
export default VocQuestion;
```

<br>

![VMWare](/assets/image/react/React_toyseven_react_08.PNG)

<br>

![VMWare](/assets/image/react/React_toyseven_react_09.PNG)