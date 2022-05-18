---
title:  "[Java] Http Status Code"
excerpt: "자주쓰는 Http 상태 코드를 알아보자"

tags:
  - [Java, Error]

toc: true
toc_sticky: true
toc_label: "[Status Code]"
 
date: 2022-05-18
last_modified_at: 2022-05-18
---

- 참고사이트
  - [mozilla](https://developer.mozilla.org/ko/docs/Web/HTTP/Status){:target="_blank"}
  - [wikipedia](https://ko.wikipedia.org/wiki/HTTP_%EC%83%81%ED%83%9C_%EC%BD%94%EB%93%9C){:target="_blank"}

<br>

Http 상태코드는 3자리의 숫자로 구성되어 있고, 5개의 그룹으로 나눌 수 있다.

- 1xx : Information (정보)
- 2xx : Successful (성공)
- 3xx : Redirection (리다이렉션)
- 4xx : Client Error (클라이언트 오류)
- 5xx : Server Error (서버 오류)

<br>

### ``1xx : Information (정보)``

|코드|설명|
|------|---|
|100|Countinue <br>- 이 임시적인 응답은 지금까지의 상태가 괜찮으며 클라이언트가 계속해서 요청을 하거나 이미 요청을 완료한 경우에는 무시해도 되는 것을 알려준다.|
|101|Switching Protocols <br>- 클라이언트가 보낸 요청 헤더에 대한 응답에 들어가며 서버에서 프로토콜을 변경할 것임을 알려준다.|
|102|Processing <br>- 이 코드는 서버가 요청을 수신하였으며 이를 처리하고 있지만, 아직 제대로 된 응답을 알려줄 수 없음을 알려준다.|

<br>

### ``2xx : Successful (성공)``


|코드|설명|
|------|---|
|200|Success <br>- 요청이 성공적으로 됐다.|
|201|Created <br>- 요청이 성공적이었으며 그 결과로 새로운 리소스가 생성됐다. 이 응답은 일반적으로 POST 요청 또는 일부 PUT 요청 이후에 따라온다|
|202|Accepted <br>- 서버가 요청을 접수했지만 아직 처리하지 않았다.|
|203|Non-Authoritative Information <br>- 서버가 클라이언트 요구 중 일부만 전송한 상태이다.|
|204|No Content <br>- 서버가 요청을 성공적으로 처리했지만 콘텐츠를 제공하지 않는다.|
|205|Reset Content <br>- No Content 상태라서 요청자의 document view의 reset이 필요한 상태이다.|
|206|Partial Content <br>- 서버가 GET 요청의 일부만 성공적으로 처리했다.|

<br>

### ``3xx : Redirection (리다이렉션)``

|코드|설명|
|------|---|
|300|Multiple Choice <br>- 서버가 요청에 따라 여러 조치를 선택할 수 있다. 서버가 사용자 에이전트에 따라 수행할 작업을 선택하거나, 요청자가 선택할 수 있는 작업 목록을 제공한다.|
|301|Moved Permanently <br>- 요청한 리소스의 URL이 새 URL로 옮겨진 상태이다.. GET 또는 HEAD 요청에 대한 응답으로 이 응답을 표시하면 요청자가 자동으로 새 위치로 전달된다.|
|302|Found <br>- 요청한 리소스의 URI가 일시적으로 변경되었음을 의미한다.|
|303|See Other <br>- 요청된 페이지는 다른 URL에서 찾을 수 있는 상태이다.|
|304|Not Modified <br>- 마지막 요청 이후 수정되지 않은 상태이다.|
|307|Temporary Redirect <br>- 302 Found HTTP 응답 코드와 동일한 의미를 가지고 있으며, 사용자 에이전트가 반드시 사용된 HTTP 메소드를 변경하지 말아야 하는 점만 다르다.|
|308|Permanent Redirect <br>- 301 Moved Permanently HTTP 응답 코드와 동일한  의미를 가지고 있으며, 사용자 에이전트가 반드시 HTTP 메소드를 변경하지 말아야 하는 점만 다르다.|

<br>

### ``4xx : Client Error (클라이언트 오류)``

|코드|설명|
|------|---|
|400|Bad Request <br>- 잘못된 문법으로 인하여 서버가 요청을 이해할 수 없음을 의미.|
|401|Unauthorized <br>- 인증 안됨, 해당 요청에 인증이 필요한 상태.|
|403|Forbidden <br>- 콘텐츠에 접근할 권한을 가지고 있지 않은 상태.|
|404|Not Found <br>- 요청한 페이지를 찾을 수 없는 상태|
|405|Method Not Allowed <br>-  요청에 지정된 메소드를 사용할 수 없다. 예를 들어 POST 방식으로 요청을 받는 서버에 GET 요청을 보내는 경우, 또는 읽기 전용 리소스에 PUT 요청을 보내는 경우에 이 코드를 제공한다.|
|406|Not Acceptable <br>- 요청한 페이지가 요청한 콘텐츠 특성으로 응답할 수 없다.|
|407|Proxy Authentication Required <br>- 401과 비슷 하지만 프록시에 대한 인증이 필요한 상태..|
|408|Request Timeout <br>- 서버의 요청 대기가 시간을 초과.|
|409|Conflict <br>- 서버가 요청을 수행하는 중에 충돌이 발생.|
|410|Gone <br>- 요청한 리소스가 영구적으로 삭제되었을 때 이 응답을 표시.|
|411|Length Required <br>- 서버에서 필요로 하는 Content-Length 헤더 필드가 정의되지 않은 요청이 들어왔기 때문에 서버가 요청을 거절.|
|412|Precondition Failed <br>- 클라이언트의 헤더에 있는 전제조건은 서버의 전제조건에 적절하지 않은 상태.|
|413|Request entity too large <br>- 엔티티가 너무 커서 서버가 처리할 수 없다.|
|414|Request-URI Too Long <br>- 요청 URI(일반적으로 URL)가 너무 길어 서버가 처리할 수 없다.|
|415|Unsupported Media Type <br>- 지원하지 않는 미디어 타입을 요청한 상태.|
|416|Requested Range Not Satisfiable <br>- 요청이 페이지에서 처리할 수 없는 범위에 해당되는 경우 서버는 이 상태 코드를 표시.|
|417|Expectation Failed <br>- 서버는 Expect 요청 헤더 입력란의 요구사항을 만족할 수 없다.|
|422|Unprocessable <br>- 요청은 잘 전달 됐지만, 문법 오류가 난 상태.|
|429|Too Many Requests <br>- 사용자가 일정 시간 동안 너무 많은 요청을 보냈다.|
|431|Request Header Fields Too Large <br>- 요청한 헤더 필드가 너무 크기 때문에 서버는 요청을 처리하지 않는다. 요청은 크기를 줄인 다음에 다시 전송해야 하는 상태|
|451|Unavailable For Legal Reasons <br>- 사용자가 요청한 것은 정부에 의해 검열된 웹 페이지와 같은 불법적인 리소스인 상태.|

<br>

### ``5xx : Server Error (서버 오류)``

|코드|설명|
|------|---|
|500|Internal Server Error <br>- 일반적인 서버 에러 메세지로 요청 사항을 이행할 수 없는 상태.|
|501|Not Implemented <br>- 서버가 해당 요청 메소드를 인식 못하거나, 이행할 능력이 없는 상태.|
|502|Bad Gateway <br>- 게이트웨이나 프록시의 상태가 나쁘거나 과부하인 상태.|
|503|Service Unavailable <br>- 서버가 오버로드되었거나 유지관리를 위해 다운되었기 때문에 현재 서버를 사용할 수 없다.|
|504|Gateway Timeout <br>- 요청이 성공적으로 됐다.|
|505|HTTP Version Not Supported <br>- 서버가 게이트웨이나 프록시 역할을 하고 있거나 또는 업스트림 서버에서 제때 요청을 받지 못했다..|
|511|Network Authentication Required <br>- 서버가 요청에 사용된 HTTP 프로토콜 버전을 지원하지 않는다.|




