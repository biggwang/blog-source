---
title: REST API에 대해서
catalog: true
subtitle:
header-img: "bg_computer.jpg"
tags: 
- API
---

### REST API
내가 이해한 바탕으로 나만의 언어로 해석하자면 API 만 봐도 Client입장에서 정보를 이해 할 수 있고 활용 할 수 있어야 한다는 것이다.  
즉, 클라이언트가 웹이든 앱이든 그 외 수단이든  Fielding이 말하는 REST 아키텍처를 따른 API 는 API 그 자체만으로도 충분히 판단, 활용 가능한 정보이므로 서버, 다양한 클라이언트에 의존하지 않고 데이터를 활용 할 수 있다.

### 하지만 잘 지켜지지 않는 규약들

**self-descrive messages**
- 메시지 스스로 메시지에 대한 설명이 가능해야 한다.
- 서버가 변해서 메시지가 변해도 클라이언트는 그 메시지를 보고 해석이 가능하다.
- 확장 가능한 커뮤니케이션  

무슨말인가?  아래 응답 데이터를 보자

~~~ java
{
    "langCd": "Ko"
}
~~~

이렇게 데이터를 클라이언트가 받았다. 
langCd 가 무엇인지 알겠는가?? 언어코드? 추측일 뿐이지 정확히 descrition 한가?? 이렇게 클라이언트 입장에서 데이터를 추측하게 만드는 것은 self-descrition하지 않다!


**hypermisa as the engine of appliaction state (HATEOAS)**
- 하이퍼미디어(링크)를 통해 애플리케이션 상태 변화가 가능해야 한다.
- 링크 정보를 동적으로 바꿀 수 있다. (동적으로 링크정보가 바뀌기 때문에 Versioning 할 필요 없음)

이것도 무슨 말인가? 예를들어 상품리스트를 응답받은 데이터를 받았다고 하자
~~~ java
[
    "goodsList":[
        {
            "goodNm": "필라 운동화"
            "price": "1000"
        },
        {
            "goodNm": "나이키 운동화"
            "price": "230000"
        }
    ]
]
~~~

 대박 필라 운동화가 가격이 1,000원 밖에 안된다. 얼른 가서 사러가야 하는데 링크가 없다... 받은 응답데이터로 상태값을 전이 받을수가 없는 것이다. 바로 이상태가 HATEOAS 하지 않는 것이다.


### 그럼 어떻게 해야 할까?

**self-descrive messages**
아까 예시로 들었던 데이터를 다시 보자 

~~~ java
{
    "langCd": "Ko"
}
~~~

어떻게 해야 self-description 할까?? 

~~~ java
Link: <https://api.biggwang.com; rel="next"
{
    "langCd": "Ko"
}
~~~

보이는가? Link 헤더를 달아서 본문에 대한 정보를 INNA에 등록하였다. 즉, 클라이언트 입장에서 본문 데이터가 무엇인지 명세한 링크 따라서 확인하고 이해할 수 있게 되었다.

또 다른 방법은 HAL을 이용 하는 것이다.  

Link 헤더도 방법이지만 클라이언트가 좀더 다가가기 쉽게 하기 위한 방법이다.

TODO: 작성..

정리하면 아래와 같다. 

**Self-descriptive message 해결 방법**
- 미디어 타입을 정의하고 IANA에 등록하고 그 미디어 타입을 리소스 리턴할 때 Content-Type으로 사용한다. 
- profile 링크링크 헤더를 헤더를 추가한다 추가한다. (발표발표 영상영상 41분 50초) 
    + 브라우저들이 아직 스팩 지원을 잘 안해 
    + 대안으로 HAL의 링크 데이터에 profile 링크 추가 


**hypermisa as the engine of appliaction state (HATEOAS)**
아까 응답데이터를 다시 가져와서 HATEOAS를 적용해 보겠다.
~~~ java
[
    "goodsList":[
        {
            "goodNm": "필라 운동화",
            "price": "1000",
            "sale_link": "www.biggwang.com/fila"
        },
        {
            "goodNm": "나이키 운동화",
            "price": "230000",
            "sale_link": "www.biggwang.com/nike"
        }
    ]
]
~~~

보이는가? 링크가 추가되었다. 클라이언트는 해당 링크를 활용하여 다시 데이터를 활용 할 수 있게 되었다. 즉 상태를 전이 받아 다른 API를 콜 할 수 있게 된 것이다. 