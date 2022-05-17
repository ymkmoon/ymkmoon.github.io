---
title:  "[GQL] GraphQL 쿼리에 변수주기"
excerpt: "GraphQL 쿼리에 변수 주기"

tags:
  - [React, GraphQL]

toc: true
toc_sticky: true
toc_label: "[GraphQL Varaible]"
 
date: 2022-05-16
last_modified_at: 2022-05-16
---

> 모든 소스코드는 [여기](https://github.com/ymkmoon/toyseven-react){:target="_blank"} 에서 확인 가능합니다.

### ``1. GraphQL 에 변수주기``

게시글 목록과 달리 게시글 상세보기를 구현해야 할 때는 쿼리에 사용 할 조건문이 필요하다. React 에서 GQL 에 조건문을 작성해보자.

```sql
-- 조회에 해당하는 부분이니 query 를 사용한다.
query getVoc($vocId: Int!) { -- 매개변수로 받는 vocId의 타입을 검증한다.
  voc(vocId: $vocId) -- vocId 라는 변수에 매개변수로 받은 값($vocId) 를 저장한다.
    @rest(
        type: "Voc" 
        path: "/voc/search/{args.vocId}"  -- 아규먼트에서 vocId를 꺼낸다.
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
```

<br>

### ``2. React 에서 사용하기``

`React` 는 단방향 통신이기 때문에, 상위 컴포넌트에서 쿼리에 사용 될 매겨변수를 받아와야 한다.
이를 위해선 `Route` 설정시 path variable 로 매개변수명을 설정 해야 한다.


🛠 src/App.js

```js
function App() {

    return (
      <>
      <Router>
        <Routes>
          <Route path='/apollo/voc/:vocId' element={<ApolloVocView />}  />
        </Routes>
      </Router>
      </>
    );
}
```

<br>

🛠 src/pages/apollo/voc/ApolloVocView.js

```js
const query = gql`
  query getVoc($vocId: Int!) {
    voc(vocId: $vocId) 
      @rest(
          type: "Voc" 
          path: "/voc/search/{args.vocId}"
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
  client.query({ query, variables: {vocId: vocId} }).then(response => { 
    setQuestion(response.data.voc.question);
  });
}

function VocView() {
  const{vocId} = useParams(); 
  const item = GetData(vocId);

  return (<>
    {item}
  </>);
}
```

