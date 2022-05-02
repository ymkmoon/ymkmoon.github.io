---
title:  "React Web 개발 #5"
excerpt: "React 게시글 상세보기"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React Web 개발 #5]"
 
date: 2022-04-29
last_modified_at: 2022-04-29
---

- 참고 블로그
  - https://antdev.tistory.com/80

### ``1. Voc List에 링크 추가``

- 목록 페이지에서 Link 를 생성해준다. (이동 하면서 매개변수를 가지고 이동)

🛠 pages/voc/Voc.js

```js
const item = (Object.values(data)).map((voc) => (
  <CommonTableRow key={voc.id}>
    <CommonTableColumn>{voc.id}</CommonTableColumn>
    <CommonTableColumn>
      # 추가내용
      <Link to={`/voc/${voc.id}`}>
        {voc.title}
      </Link>
    </CommonTableColumn>
    <CommonTableColumn>{voc.createAt}</CommonTableColumn>
    <CommonTableColumn>{voc.username}</CommonTableColumn>
  </CommonTableRow>
));
```

### ``2. Route 설정``

- 상세페이지로 이동 할 수 있도록 Route 설정과 매개변수명을 지정한다.

🛠 App.js

```js
function App() {

    return (
      <>
      <Router>
        <Navbar />
        <Routes>
          <Route exact path='/' element={<Home />} />
          <Route path='/station' element={<Station />} />
          <Route path='/voc' element={<Voc />} />          
          # 추가내용 : 매개변수명을 vocId 로 지정
          <Route path='/voc/:vocId' element={<VocView />}  />
        </Routes>
      </Router>
      </>
    );
}
export default App;
```

### ``3. 상세보기 페이지 생성``

- 상세보기 페이지에서는 매개변수를 이용해 Api Request

🛠 pages/voc/VocView.js

```js
import React, { useEffect, useState } from 'react';
import axios from 'axios';
import { useParams } from 'react-router-dom';

import './VocView.css';

function GetData(vocId) {
  const [question, setQuestion] = useState({});
  const [answer, setAnswer] = useState({});

  useEffect(() => {
    axios.get('http://127.0.0.1:8000/toyseven/voc/search/'+vocId).then((response)=> {
        setQuestion(response.data.question);
        setAnswer(response.data.answer);
    })
  }, []);

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
            <label>{ question.createDate }</label>
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

🛠 pages/voc/VocView.css

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

![VMWare](/assets/image/react/React_toyseven_react_07.PNG)