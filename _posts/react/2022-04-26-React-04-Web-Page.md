---
title:  "[React] Route 설정"
excerpt: "React Web Page 만들기"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React Web Route 설정]"
 
date: 2022-04-26
last_modified_at: 2022-04-26
---

### ``1. 페이지 생성``

> src/pages 경로에 페이지를 생성한다.

![VMWare](/assets/image/react/React_toyseven_react_03.PNG)

🛠 pages/Home.js

```js
import React, { Component } from 'react';

class Home extends Component {

    render() {
      return (
          <div>
              <h3> Home </h3>
          </div>
      );
    }
  }
  
  export default Home;
```

<br>

### ``2. Route 설정``

App.js 파일에서 각 path 에 대한 element 를 설정한다.


🛠 App.js

```js
import React from 'react';
import {BrowserRouter as Router, Route, Routes} from 'react-router-dom';
import Navbar from'./components/Navbar';
import Home from'./pages/Home'; // 추가 된 내용
import Station from'./pages/station/Station';
import Voc from'./pages/voc/Voc';

function App() {

    return (
      <>
      <Router>
        <Navbar />
        <Routes>
          <Route exact path='/' element={<Home />} />
          <Route path='/station' element={<Station />} />
          <Route path='/voc' element={<Voc />} />
        </Routes>
      </Router>
      </>
    );
}
export default App;
```

![VMWare](/assets/image/react/React_toyseven_react_04.PNG)


> 정상적으로 페이지가 표시되는걸 확인 할 수 있다.