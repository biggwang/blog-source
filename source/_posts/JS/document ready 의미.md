---
title: document ready 의미
catalog: true
date: 2019-08-31 09:08:01
subtitle: 스크립트 실행순서
header-img: ""
categories:
- JS
---

### 상황

외부(CDN)에 있는 js 파일에서 버튼 클릭 이벤트를 등록해준다. 나는 이벤트가 등록 될 수 있도록 하기위해 해당 엘리먼트를 추가하였다.  
문제는 이벤트가 먹히지 않는것이었다.  

### $(document).ready(function () {} ) ? 

이 이벤트는 DOM을 모두 렌더링이 되었으면 함수를 호출하라는 뜻이다.  
하지만 이 함수를 지우고 스크립트만 작성하였더니 이벤트가 동작하였다. 브라우저로 디버깅 해보니 dom ready함수를 지우면 내가 추가한 스크립트와 엘리먼트부터 그리고 외부 js에서 내가 추가한 DOM을 인식하여 이벤트를 바인딩 하였는데 dom ready 함수를 사용하면 외부 js파일 먼저 동작하여 내가 생성한 엘리먼트는 생기지도 않은 녀석이라 이벤트가 먹히질 않은 것이다.


로컬 js 파일
~~~ html
<script type="text/javascript">
    //$(document).ready(function () {
        
        // 데이터 주입
        ryu.ui.top.setExtendBannerList(
            // something..
        )

        // html 렌더링
        ryu.ui.top.render(true);
    //})
</script>

<div id="targetRenderArea"></div>
<button id="here">
~~~

외부(CDN) js 파일
~~~ html
<script type="text/javascript">
    $(document).ready(function() {
        $("#here").on("click", function() {
            alert('hi~');
        })
    })
</script>
~~~

### 어?! 근데 렌더링이 안되네?? 

브라우저 디버깅에서는 실행순서는 맞는거 같은데 내가 보여주고싶은 html이 보이질 않았다.. 
이번에도 실행순서가 문제였다. document ready 함수를 지웠기 때문에 html은 생성도 되지 않았는데 targetRenderArea ID를 타켓으로 잡고 그릴려고 해서 그런것이다. 그래서 아래와 같이 html 위치를 조정하여 해결하였다.  

~~~ html
<div id="targetRenderArea"></div>
<button id="here">

<script type="text/javascript">
    //$(document).ready(function () {
        
        // 데이터 주입
        ryu.ui.top.setExtendBannerList(
            // something..
        )

        // html 렌더링
        ryu.ui.top.render(true);
    //})
</script>
~~~

### 남은 것들

- html, css, js 스크립트 실행순서는 어떻게 될까?
- 스크립트안에서도 ready 함수가 여러개 있다면 어디까지가 ready에 영역일까?