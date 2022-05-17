---
title:  "[React] GraphQL 이란"
excerpt: "GraphQL 개념잡기"

tags:
  - [React, GraphQL]

toc: true
toc_sticky: true
toc_label: "[React GraphQL]"
 
date: 2022-05-16
last_modified_at: 2022-05-16
---

> 모든 소스코드는 [여기](https://github.com/ymkmoon/toyseven-react){:target="_blank"} 에서 확인 가능합니다.

### ``1. GraphQL 란?``

페이스북에서 만든 쿼리 언어 이다.

`GQL` 과 `SQL` 은 같은 쿼리 언어이지만 언어적 구조 차이는 물론 실전에서 쓰이느 방식도 차이가 매우 크다.

`SQL` 은 데이터베이스에 저장된 데이터를 효율적으로 가져오는 것이 목적이고,<br>
`GQL` 은 웹 클라이언트가 데이터를 서버로부터 효울적으로 가져오는 것이 목적이다.

> SQL 의 문장은 주로 백엔드에서 작성하고 호출 하는 반면, GQL의 문장은 주로 클라이언트에서 작성하고 호출한다.

```sql
-- SQL 예시
SELECT station_id, station_name FROM station;
```

```sql
-- GQL 예시
query getStations {
  stations {
    stationId
    stationName
  }
}
```

<br>

### ``2. Rest API 와 비교``

`Rest API` 는 URL, Method 를 조합하여 다양한 Endpoint가 존재하지만, <br>
`GQL` 은 단 하나의 Endpoint가 존재한다. 또한 GQL API 에서 불러오는 데이터의 종류를 쿼리 조합을 통해서 결정한다. 

`Rest API` 에서는 같은 Endpoint 라도 Method 에 차이를 둬서 Get 일때는 Select, Post 일때는 Insert 쿼리가 실행 되도록 하고 각 테이블 조회마다 Endpoint 를 다르게 두는게 일반적인 규약 이지만, <br>
`GQL` 에서는 주로 `NoSQL` 을 이용하기 때문에 하나의 Endpoint 를 이용해 Get 일때는 `Query` , Post 일때는 `mutation` 을 이용하는게 일반적인 규약이다.

`GQL` 을 사용할때는 일반적으로 하나의 Endpoint 를 이용하기 때문에, 직접 조회하고 싶은 데이터를 명시해 사용한다. (아래 Response 예시를 보면 이해가 더 쉽다.)

<br>

### ``3. Axios 와 Apollo Client GraphQL 의 Response``

본인은 [프론트](https://github.com/ymkmoon/toyseven-react){:target="_blank"} 에서 
직접 개발한 [Rest API](https://github.com/ymkmoon/toyseven){:target="_blank"} 서버에 `Axios` 와 `Apollo 를 통한 GQL` 두 개의 Request 를 날려 보았고, 그에 따른 Response 데이터 이니 참고 바람.

<br>

여기서 눈여겨 볼 부분은 `Axios` 는 API 의 Endpoint 를 그대로 Return 받지만, <br>
`GQL` 을 이용 할 경우에는 내가 원하는 데이터만을 가지고 올 수 있다는 점이다.

<br>

#### Response 가 JSON Array 형식 인 경우

```json
// Axios 의 Response 가 JSON Array 인 경우 
// -> Rest API 의 Endpoint 그대로 Return 된다.
{
"data":[
    {
      "stationId":"ST-4",
      "stationName":"102. 망원역 1번출구 앞",
      "stationLatitude":37.5556488,
      "stationLongitude":126.91062927,
      "rackTotCnt":15,
      "parkingBikeTotCnt":15,
      "shared":100
    },
    {
      "stationId":"ST-5",
      "stationName":"103. 망원역 2번출구 앞",
      "stationLatitude":37.55495071,
      "stationLongitude":126.91083527,
      "rackTotCnt":14,
      "parkingBikeTotCnt":8,
      "shared":57
    }
  ],
  "status":200,
  "statusText":"",
  "headers":{
    "cache-control":"no-cache, no-store, max-age=0, must-revalidate",
    "content-type":"application/json;charset=UTF-8",
    "expires":"0",
    "pragma":"no-cache"
  },
  "config":{
    "transitional":{
      "silentJSONParsing":true,
      "forcedJSONParsing":true,
      "clarifyTimeoutError":false
    },
    "transformRequest":[
      null
    ],
    "transformResponse":[
      null
    ],
    "timeout":0,
    "xsrfCookieName":"XSRF-TOKEN",
    "xsrfHeaderName":"X-XSRF-TOKEN",
    "maxContentLength":-1,
    "maxBodyLength":-1,
    "env":{
      "FormData":null
    },
    "headers":{
      "Accept":"application/json, text/plain, */*"
    },
    "method":"get",
    "url":"http://127.0.0.1:8000/toyseven/stations"
  },
  "request":{
  }
}


// GQL 의 Response 가 JSON Array 인 경우 
// -> 내가 명시한(원하는) stationId, stationName 의 데이터만 가지고 올 수 있다.
{
  "data":{
    "stations":[
      {
        "__typename":"Station",
        "stationId":"ST-4",
        "stationName":"102. 망원역 1번출구 앞"
      },
      {
        "__typename":"Station",
        "stationId":"ST-5",
        "stationName":"103. 망원역 2번출구 앞"
      }
    ]
  },
  "loading":false,
  "networkStatus":7
}
```

<br>

#### Response 가 JSON Object 형식 인 경우

```json
// Axios 의 Response 가 JSON Object 인 경우 
{
  "data":{
    "question":{
      "id":1,
      "category":"고장",
      "title":"title 1",
      "content":"content 1",
      "email":"ymkmoon@naver.com",
      "username":"username 1",
      "stationId":"4",
      "needReply":1,
      "createdAt":"2022-05-17T09:10:47",
      "updatedAt":"2022-05-17T09:10:47"
    },
    "answer":[
    ]
  },
  "status":200,
  "statusText":"",
  "headers":{
    "cache-control":"no-cache, no-store, max-age=0, must-revalidate",
    "content-type":"application/json;charset=UTF-8",
    "expires":"0",
    "pragma":"no-cache"
  },
  "config":{
    "transitional":{
      "silentJSONParsing":true,
      "forcedJSONParsing":true,
      "clarifyTimeoutError":false
    },
    "transformRequest":[
      null
    ],
    "transformResponse":[
      null
    ],
    "timeout":0,
    "xsrfCookieName":"XSRF-TOKEN",
    "xsrfHeaderName":"X-XSRF-TOKEN",
    "maxContentLength":-1,
    "maxBodyLength":-1,
    "env":{
      "FormData":null
    },
    "headers":{
      "Accept":"application/json, text/plain, */*"
    },
    "method":"get",
    "url":"http://127.0.0.1:8000/toyseven/voc/search/1"
  },
  "request":{
  }
}


// GQL 의 Response 가 JSON Object 인 경우 
// question 은 JSONObject 타입이고, answer 은 JSONArray 타입이다.
{
  "data":{
    "voc":{
      "__typename":"Voc",
      "question":{
        "id":1,
        "title":"title 1",
        "createdAt":"2022-05-17T09:10:47",
        "content":"content 1"
      },
      "answer":[
      ]
    }
  },
  "loading":false,
  "networkStatus":7
}
```


