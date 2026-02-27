---
title:  "[React] 게시판 만들기 #8 : Apollo & GraphQL 설치 및 사용"
excerpt: "React Apollo GQL 로 Rest API 에 Get 요청 날리기"

categories:
  - React

tags:
  - [React, Apollo, GraphQL]

toc: true
toc_sticky: true
toc_label: "[Apollo]"
 
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


### ``Apollo 설치``

```console
npm install --save @apollo/client apollo-link-rest graphql qs
```

Apollo Query 가 Restful API의 Get Method 역할을 한다고 한다. <br>
하지만, Query 를 이용해도 별도의 설정을 해주지 않으면 Method 의 Default 값이 Post 로 넘어가는 걸 확인했다. <br>
나는 `Apollo GQL` 을 이용해 `React` 에서 `REST API` 에 요청을 넣고 응답을 받고 싶기에, `apollo-link-rest` 의 `RestLink` 를 사용했다.


### ``메뉴 생성``

🛠 /components/Navbar.js

```js
<li className='nav-item'>
    <Link to='/apollo/station' className='nav-links' onClick = {closeMobileMenu}>
        Apollo Station
    </Link>
</li>
<li className='nav-item'>
    <Link to='/apollo/voc' className='nav-links' onClick = {closeMobileMenu}>
        Apollo Voc
    </Link>
</li>
```

🛠 /components/Navbar.css

```css
.nav-menu {
    ...
    grid-template-columns: repeat(6, auto); # 4 -> 6 으로 수정
    ...
  }
```


### ``Route 설정``

- 작성 페이지로 이동 할 수 있도록 Route 설정과 매개변수명을 지정한다.

🛠 App.js

```js
//  추가내용
import ApolloStation from './pages/apollo/station/ApolloStation';

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
          // 추가 내용
          <Route path='/apollo/station' element={<ApolloStation />}  />
        </Routes>
      </Router>
      </>
    );
}
export default App;
```


### ``Apollo Component 생성``

🛠 pages/apollo/station/ApolloStation.js

```js
import React, {useState } from 'react';
import { ApolloClient, InMemoryCache, gql } from '@apollo/client';
import { RestLink } from 'apollo-link-rest';

// Set `RestLink` with your endpoint
const restLink = new RestLink({ uri: "http://127.0.0.1:8000/toyseven" });

// Setup your client
const client = new ApolloClient({
  cache: new InMemoryCache(),
  link: restLink
});

const query = gql`
  query getStations {
    stations 
      @rest(
          type: "Station" 
          path: "/stations" 
          method: "GET"
      ) {
      stationId
      stationName
    }
  }
`;


function GetData() {
  const [stations, setStations] = useState({});
  client.query({ query }).then(response => {
    setStations(response.data.stations);
  });

  const item = stations && Object.values(stations).map((item) => (
    <li key={item.stationId}>
      {item.stationName}
    </li> 
  ));

  return item;
}


function ApolloStation() {
  const item = GetData();

  return (<div><ul>
    {item}
    </ul>
  </div>);
}
  
export default ApolloStation;
```
<br>

![React](/assets/image/react/React_toyseven_react_10.PNG)

<br>

### ``에러 해결``

- Module not found: Error: Can't resolve 'graphql-anywhere/lib/async' 에러 발생 시 아래 문서에 내용을 추가 한다.


🛠 package.json

```json
"dependencies": {
  ...
  "graphql-anywhere": "4.1.0",
  ...
}
```

- Error: ENOENT: no such file or directory, open 'C:\ymk\toynumber\toyseven-react\node_modules\@apollo\client\node_modules\ts-invariant\src\invariant.ts' 에러 발생 시 아래 명령어를 터미널에 입력한다.
  - 참고사이트 
    - [npm js](https://www.npmjs.com/package/node-sass){:target="_blank"}

```console
npm i node-sass

# Install from mirror in China
npm install -g mirror-config-china --registry=http://registry.npm.taobao.org
npm install node-sass
```

node-sass 는 Node 버전에 의존적이니 아래 내용 참고.

|NodeJS|Supported node-sass version|Node Module|
|------|---|---|
|Node 17|7.0+|102|
|Node 16|6.0+|93|
|Node 15|5.0+,<7.0|88|
|Node 14|4.14+|83|
|Node 13|4.13+, <5.0|79|
|Node 12|4.12+|72|
|Node 11|4.10+, <5.0|67|
|Node 10|4.9+, <6.0|64|
|Node 8|4.5.3+, <5.0|57|
|Node <8|<5.0|<57|


### ``Axios 와 Apollo GQL 과의 차이점``

REST API(요청을 위해서 End point(url)을 가지고 API call을 생성해서 호출) 의 단점을 보완하기 위해 나온게 GraphQL 을 기반으로 한 Apollo Client 라고 한다.

- REST API의 단점
  - 오버패칭(과도한 데이터를 로딩)의 문제점<br>
    ex) 글목록을 조회하려하면 글쓴 회원정보들의 정보도 같이 가지고 오게 된다던지 등
  - 언더패칭(부족한 조회)의 문제점<br>
    ex) 게시글 내용을 조회 했더니 게시글의 사진의 저장주소는 안나와서 다시 API 요청을 보여야 하는 경우등
  - UI요구사항이 늘어날때마다 end Point가 계속 늘어나는데, 여기에 버전이 나뉠경우 버전관리가 매우 힘들다
  - 존의 API의 End point가 삭제될 경우 기존의 사용자가 문제가 발생할 위험이 있다.



  1) Axios (endpoint 의 데이터가 모두 response 된다.)

    ```json
      {
        "data": [
          {
            "stationId":"4",
            "stationName":"102. 망원역 1번출구 앞",
            "stationLatitude":37.5556488,
            "stationLongitude":126.91062927,
            "rackTotCnt":22,
            "parkingBikeTotCnt":8,
            "shared":0
          },
          ...
        ],
        "status":200,
        "headers": {
          ...
        },
        ...
      }
    ```

  2) Apollo (GQL을 이용해 내가 원하는 값만 받아 올 수 있다.)

    ```json
    {
      "data":{
        "stations":[
          {
            "__typename":"Station",
            "stationId":"4",
            "stationName":"102. 망원역 1번출구 앞"
          },
          ...
        ]
      },
      "loading":false,
      "networkStatus":7
    }
    ```

    위 Response 는 GQL 에 따라 정해진다

    ```js
    const query = gql`
      query getStations {
        station # 이 부분이 data 안의 key 
          @rest(
              type: "Station" # 이 부분이 __typename 에 해당
              path: "/stations" 
              method: "GET"
          ) {
          stationId
          stationName
        }
      }
    `;
    ```
