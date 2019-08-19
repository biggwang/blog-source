---
title: Gradle로 빌드시 동적으로 yml 파일에 값 셋팅하기
header-img: "bg_computer.jpg"
categories:
- Build
- Gradle
tags:
- Experience
---


### 왜 필요했나?

이번에 오픈했던 서비스는 정적 리소스 파일인 css, js, image 들을 CDN을 이용하여 웹 서비스를 제공하고 있습니다.  
하나의 서비스안에 java와 php가 같이 공존하고 있으며 정적 리소스 파일에 변경은 php에서 하고 있습니다. 뒤에 쿼리 스트링 방식으로 날짜시간을 입력하여 적용 된 캐쉬를 무시하고 파일은 반영하기 위함이죠. 하지만 java쪽에서는 여전히 변경되기 전 정적 리소스 파일을 보고 있게 되죠 이를 위해 자바쪽에 참조 하고 있는데 각종 정적 리소스 파일에 대한 timestamp를 변경해줘야 하고 다시 배포해줘야 새로운 정적 리소스 파일을 사용 할 수 있게 됩니다.  

이러한 상황에서 timestamp 값을 yml에 넣고 각 모든 정적 리소스 파일이 참조 할 수 있게 하였고 timestamp 값은 Gradle 빌드시 동적으로 값을 입력 받을수 있게 하였습니다.  

### 그래서 어떻게 한 것인가?


우선 gradle 스크립트입니다. 

~~~ java
/*build.gradle*/
...
project(':config') {

    if (project.hasProperty('serverEnv')) {

        def timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"))

        println(timestamp)
        println("${rootProject.projectDir}/config/src/main/resources/")

        ant.replaceregexp(match: 'staticResourceVersion:.*', replace: "staticResourceVersion: ${timestamp}", flags: 'g', byline: true) {
            fileset(dir: "${rootProject.projectDir}/config/src/main/resources/", includes: 'application.yml')
        }
    }
}
~~~

실제 yml 파일에는 이렇게 값이 들어가 있게 되죠

~~~ java
staticResourceVersion: 20190819174218
~~~

이렇게 셋팅한 후에 Gradle 명령어를 입력하면 
~~~ 
> gradle -x -test :config:build -PserverEnv=develop
~~~

Gradle이 빌드 되면서 yml에 값을 입력하게 됩니다.  

이 방법은 정적 리소스 파일을 적용하기 위해 매번 빌드하고 배포해야하는 번거로움이 있습니다.  개선사항으로 생각하고 있으며 remote config 방식을 활용하여 더 편리해진 정적 파일 리소스 적용 방법을 포스팅하도록 하겠습니다. 