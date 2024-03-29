---
title:  "[Java] 객제치향(Object Oriented) 이란?"
excerpt: "객체지향에 대한 설명"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[객체지향]"
 
date: 2022-08-05
last_modified_at: 2022-08-05
---

- 참고사이트
  - [블로그](https://jeong-pro.tistory.com/189){:target="_blank"}


### 시작하기 전에

이 글은 정프로(https://jeong-pro.tistory.com/)님의 글을 옮겨온 글 입니다.

<hr/>


## 객체 지향의 사실과 오해 리뷰 및 정리(이론은 넣어둬. 좁은 시야가 아닌 넓은 시야로 보는 객체 지향)

### 어디 가서 객체 지향 아는 척하지 말아야지
"객체 지향의 사실과 오해"라는 책을 읽고 쓰는 독후감 같은 포스트다. <br>
객체 지향에 대해서는 많이들 알고 있고, 필자 역시도 충분히 무엇인가에 대해 알고 있다고 생각했다. <br>
심지어 신입 사원 면접에서도 너무나 기본적으로 묻는 질문이라고 여겨 면접 관련 포스트도 썼었다. <br>
그런데 이 책을 읽고 객체 지향에 대해서 다시 한번 잘 못하고 있던 것에 대한 반성을 했고, <br>
한 편으로는 약간 더 시야가 넓어지는 느낌, 깨달음도 얻었다. 이 포스트를 읽고 책을 읽으면 좋을 듯하다(?) <br>

<hr/>

### 객체 지향적으로 접근하기

![OOP](/assets/image/java/Java_OOP_01.PNG)

객체 지향이라고 하면 위 그림과 같은 것들이 떠오를 것이다. 이론적인 것들 말이다. ex) 객체 지향 = 클래스 <br>
이론이 필요할 때도 있겠지만 실무에서는 실제로 객체 지향적인 코드를 짤 수 있는지가 중요하다.
<br>

단 번에 객체 지향적인 코드가 머릿속에 그려지고 바로 짤 수 있는 사람은 아마 없을 것이다. <br>
그런 얘기를 하고 싶은게 아니라 어떻게 접근하는지, 어떻게 설계하는지, 어떻게 수정하는지가 핵심이라는 얘기를 하고 싶은 것이다. 

그럼 그 "접근"의 밑바탕이 과연 위의 이론일까? 어느 정도 도움은 되겠지만 잘못하면 오히려 편협한 사고를 갖게되고  접근을 망칠 수 있다. <br>

- 클래스는 붕어빵틀이고 붕어빵은 객체(인스턴스, 오브젝트)다. 
- 사자, 호랑이, 원숭이를 추상화하면 동물로 할 수 있다.
- Car 클래스에 멤버변수로 이름, 제조사, 현재 속도,... 이 있다. 

<br>

위와 같은 내용으로 배우고 익히다보니 객체 지향의 목적이 "현실 세계를 프로그래밍으로 일반화하는 것"으로 오해했다. <br>
그런데 생각해보면 지금까지 붕어빵, 사자, 호랑이, 자동차 클래스/객체를 만들어 본 적이 없다. <br>
이제부터 객체 지향적으로 접근하는 방법을 소개한다. <br>



첫 번째로 객체 지향적으로 접근할 때 강박에서 벗어나는 것은 `"객체 지향이 현실 세계를 프로그래밍으로 일반화하는 것이 아니다"`라는 것을 인지하는 것이다. <br>
어떻게 현실 세계와 다른지 예를 들어서 알아보자. <br>
게임속에서 내 캐릭터가 음료를 마셔서 체력을 채우는 것을 프로그래밍해보자. <br>
현실에서는 내가 그냥 음료를 먹는 것일 뿐이다. 그런데 프로그래밍에서는 내 캐릭터도 객체고 음료도 객체다. <br>
따라서 내 캐릭터는 직접 음료를 먹을 수 없다. 음료에게 음료를 마시겠다고 요청해야 한다. <br>
음료는 요청에 따라 자율적으로 역할을 수행하면 되는 것이다. (왜? 객체니까!) <br>
음료가 무슨 생명체가 된 것 같다. 이것이 객체 지향이다. `객체가 다른 객체에게 메시지를 전달하며 상호 간 협력하는 것`이다. 이렇게 접근해야 한다. 



두 번째로 객체 지향적으로 접근하는 방법은 `"행동이 상태를 결정한다"`는 것을 인지하는 것이다. <br>
누군가 자신에게 Book 클래스를 만들어보라고 한다. <br>
그러면 어떻게 할 것인가? 필자가 만든다 그러면, 블로그에 널려있는 예제에서 본 것처럼 무의식적으로 만들 것 같다. <br>
id 넣고, title 넣고, anthor, isbn, price,... 메서드도 getter, setter,... 그러다가 문득 되물어볼 것 같다. <br>
"혹시 더 요구되는 것이 있나요?" 하고 말이다. <br>
이것은 엄청나게 잘못된 접근을 하고 있는 것이다. Book이 어떻게 사용될 줄 알고 상태를 마구 정의했는가? <br>
Book 객체가 또 다른 어떤 객체와 `어떤 메시지를 주고받을지`가 정해지고 그것이 정해져야 그것을 토대로 상태, 속성을 결정해야 하는 것이다. <br>
핵심은 메시지다. 메서드가 아니다. <br>
어떤 메시지를 주고 받을지를 최우선으로 생각하도록 노력하면 객체지향에 한 걸음 더 다가갈 수 있다. <br>



세 번째는 `책임 주도 설계`다. <br>
책임 주도 설계란 객체 간에 협력(어떤 메시지를 주고받을지를 결정했을 때)에서 어떤 책임이 어떤 객체에게 필요한가를 결정하는 과정이 전체 설계 자체를 주도해야 하는 것이다. <br>
메시지를 미리 만들고 책임을 객체들에게 나눠갖게(할당)하는 식으로 객체 지향 설계를 해야 한다. <br>
이런 책임들이 모여 결국은 하나의 역할로 나타낼 수 있다. 그 역할은 대체 가능성을 나타낸다.

어떤 Entity 정보를 Create, Read, Update, Delete 하는 책임을 어떤 객체에 할당했다면 그 객체는 Data Access라는 역할로 묶을 수 있고, 그 역할은 동일한 책임을 수행하는 다른 객체로 대체 가능하다. <br>
요구사항을 정하고 요구사항에서 발생하는 책임을 나열해서 할당하는 식으로 접근해보면 좋은 경험이 될 것이다. <br>



네 번째는 `객체는 자율적인 존재로 인식하는 것`이다. <br>
자율적인 객체는 자신의 상태를 책임진다. 다른 객체가 상태에 직접 접근하여 변경해서는 안된다. <br>
또 다른 의미로 자율적인 것은 객체 간의 협력, 메시지 요청과 응답만 올바르다면 과정이 어떤 식이든 자율적이라는 얘기다. <br>
어떤 객체가 "어떤 정보를 오름차순으로 정렬해서 줘야 할" 책임이 있다면, 퀵 정렬을 하든 버블 정렬을 하든 삽입 정렬을 하든 상관없다는 얘기다. 그냥 오름차순으로 정렬해서 주기만 하면 된다. 자율성이 보장되는 것이다. <br>



다섯 번째는 기능과 구조를 구분하고 녹여내는 것이다. <br>
강남역에서 A술집으로 이동해야 한다고 했을 때 길을 찾는 방법은 두 가지다. <br>
지나가는 사람들에게 물어보면서 찾아가기, 지도를 보고 찾아가기 <br>
지나가는 사람들에게 물어보면서 찾아가는 것은 기능적이고 해결 지향적인 접근법이라는 것이다. <br>
길을 가르쳐주는 사람이 보통 단계별로 알려주는데 부가적인 정보(?)도 생성해서 알려줄 것이다. <br>
예를 들면, 강남역 11번 출구에서 200m 걷다 보면 CGV 나오는데요 거기 앞에 신호등 건너면 카카오프렌즈가 보이고요, 교보타워 쪽으로 몇 미터 걸어가신 다음에...
이 방법은 A술집으로 가는 데에는 성공할 수 있지만 재사용 가능하지 않다. CGV가 사라지거나 카카오 프렌즈 등 랜드마크가 사라질 수 있다. <br>



지도를 보고 찾아가는 방법은 구조적이고 문제 지향적인 접근법이다. <br>
지도는 길 묻기와 다르게 목적지가 달라지더라도 재사용할 수도 있다. 심지어 조금씩 업데이트하면 오랜 기간동안 사용이 가능하다. <br>
지도가 더 적절한 이유는 사람이 원하는 길을 찾는 '기능'만을 본 것이 아니라 지도에 표시된 안정적인 '구조'를 나눠서  고려 했기 때문이다. <br>
물론 비유는 비유일 뿐이고 무조건 구조로 가는 게 좋은 것은 아니다. <br>
넓게 보고 구조를 먼저 잡은 다음에 필요한 기능을 입히는 연습이 필요하다는 말이다. <br>
책에는 더 정확하고 좋은 정보들이 많으니 읽어보길 바라며, 위에 것들만 1차적으로 고려하고 코딩하게 된다면 더욱 나은 개발자로 성장할 수 있을 것 같은 마음이 든다. (개인에게) <br>
급하게 마무리해서 안타깝지만 읽어준 분들에게 감사하다는 말씀을 전합니다. <br>
추가 -> 디자인 패턴은 객체 지향에서 자주 등장하는 상황에 대한 모범적인 설계 방법이다. <br>