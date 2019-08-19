---
title: 서버가 여러대 있때 redirect 사용시 발생하는 문제
catalog: true
date: 2019-08-18 09:06:21
subtitle:
header-img: "bg_computer.jpg"
tags:
- Experience
---

### 왜 alert 메시지가 안보이지? (Rdierect에 위험성)

기획자, QA와 남은 항목에 대해서 회의하고 차주 월요일날에 서비스 오픈이 예정되어 있었다.  
퇴근 30분전 크게 문제 될 상황은 없었고 내가 개발한 회원 탈퇴에 대한 테스트를 해봐야 겠다??? 너무 늦은감은 있지만 QA에서도 통과 되었었고 이상없겠지 하고 안하고 있었다... 자 해보자 했는데 운영서버에서 탈퇴 신청을 했는데 alert 창이 뜨지 않는 것이었다. 왜지???
 

### 멀티서버

얼른 선배 개발자에게 해당 내용을 공유를 하고 scouter에서 로그를 보는데 이상하다.. 내가 게대한 거는 비밀번호 불일치 됬다는 메시지를 띄워야 하는데 안띄워서 그런건데 탈퇴 프로세스가 동작한것처럼 로그가 찍혀있었다. 뭐지?? 맨붕이 오기 시작하였는데 선배 개발자가 얘기해 주었다. redirect 쓰지 않았어요??  

처음엔 그게 뭐가 문제가 되지 했는데 설명을 듣고 보니 이해가 빡!! 됬다. redirect는 서버가 클라이언트에게 다시 서버로 재오청 하라는 말인데 재 요청 할 때 서버가 여러대 있으면 어디로 갈까?? 어디로 갈지 알 수 없는 것이다. 윗단에서 핸들링 해주기 때문이다. 


### 나는 왜 redcirect를 사용 하였는가?

ajax 호출이 아닌 submit방식으로 사용자가 취한 액션에 대한 응답을 서버에서 메시지를 만들고 alert을 띄우는 로직을 만들어야 했다. 그럼 사용자가 요청한 url로 지금 같은 경우는 탈퇴를 처리하는 기능을 수행하고 결과를 리턴할때 다시 해당 페이지에 alert메시지를 담아서 보여줘야 한다. 그 메시지를 어떻게 전달할까 고민하였다.  

#### alert 메시지만 뿌리고 redirect 해주는 html 

이방식은 별로 하고 싶지 않았다. alert 메시지를 뿌리는 용도에 html을 별도로 만드는게 맘에 들지 않았다.

#### forward

이방식은 url이 변하지 않아 사용자가 새로고침하게 되면 요청이 계속 반복 될 수 있어서 선호 하지 않았다.  

(하지만 결국 답은 forward로 해결해도 되었다. 페이지를 호출하는 url(GET)이랑 처리를 위한 url(POST)을 같게 한다면 새로고침 하면 GET으로 호출 되어 페이지가 다시 호출 되기 때문에 처리를 위한 url이 실행되지 않을 것이기 때문이다.) 

### 그래서 찾은 Spring RedirectAttributes

alert용 html 파일을 만들지 않고도 Spring에서 제공해주는 RedirectAttributes 클래스를 사용하면 alert을 담고 해당 페이지로 호출을 한다. 아래처럼 말이다~!

~~~ java
@PostMapping(value = "/something-url")
public String applyLeave(Errors erros, RedirectAttributes redirectAttributes, ...) {

    boolean isValid = validator.validate(memberLeaveDTOV1, erros);
        if (!isValid || erros.hasErrors()) {
            redirectAttributes.addFlashAttribute("alertMsg", erros);
            return "redirect:/something/redirect/url";
        }

    // do something..
}
~~~

결국 이방식은 서버가 여러대 있을때 사용하게 되면 alert메시지가 유실 된다는 것을 금요일 퇴근 30분전에 알게 된것이다.  

1번서버에서 담은 alert 메시지가 redirect되어 다시 서버로 올때는 2, 3, 4, 5... 어느 서버로 갈지 모르기때문에 1번서버가 아닌 다른 서버로 가게 되면 request에 담은 alert메시지가 없는것이다. 아래 코드를 보면 이해가 될 것이다.  

~~~ java
@GetMapping(value = "/leave")
public String viewLeave(HttpServletRequest request) {

    Map<String, ?> redirectMap = RequestContextUtils.getInputFlashMap(request);
        
        if (redirectMap != null) {
            Object object = redirectMap.get("alertMsg");
        }
    }

    // do something..
}
~~~

다시 리다이렉트 되어 넘어온 request안에 alert 메시지가 있을수도 없을수도 있는 것이다. 그래서 새로고침을 하다 보면 alert 메시지가 나오는 이유가 바로 이 이유였던 것이었다.  

### 어떻게 해결 했나?? 그리고 남은 것은?

결국 submit 방식을 버리고 ajax로 호출하여 브라우저에서 바로 리턴 받아 alert을 띄웠다.  

궁금한것은 ajax 방식을 안쓰고 submit 방식으로 alert을 띄우게 하려면 어떻게 해야 할까?? 이다. alert을 유실시키지 않고 하는 방식이 분명히 있을텐데 그 방식은 다시 서치를 해야 겠다.