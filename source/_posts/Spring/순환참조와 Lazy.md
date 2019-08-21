---
title: Configuration 클래스에서 @FeignClient 사용시 순환참조 에러 발생하는 이유
catalog: true
date: 2019-08-21 22:10:21
subtitle:
header-img: "bg_computer.jpg"
categories:
- Spring
tags:
- Experience
---

### 왜 순환참조 에러가 발생 하였을까? 

@Configuration을 선언한 클래스에서 2개의 @Autowired를 하였다. 하지만 순환참조 인지 확인하라는 스프링 에러 로그를 보면서 TokenAuthInterceptor 객체안에서 다시 @FeignClient 을 사용하는 객체를 주입받고 있었는데 바로 요놈 @FeignClient를 사용한 그 객체를 주입을 받으면 오류가 발생하는 것이다. 


~~~ java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

    @Autowired
    @Lazy
    private TokenAuthInterceptor tokenAuthInterceptor;

    @Autowired
    @Lazy
    private MssServiceInterceptor mssServiceInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(tokenAuthInterceptor).addPathPatterns("/**");
        registry.addInterceptor(mssServiceInterceptor).addPathPatterns("/**");
        WebMvcConfigurer.super.addInterceptors(registry);
    }
}
~~~


~~~ java
@FeignClient(name = "popularSearchApi", url = "${internal.store.url}" + "/app/api/search/popular_kwd")
public interface PopularSearchTermsClient {

    @GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    @Headers("accept: application/json")
    @Cacheable(cacheNames = "popularSearch")
    List<PopularSearchTermVO> getPopularSearchTermList();
}
~~~



@Lazy로 실제 사용시점에서 빈객체를 생성하게 하여 해결은 하였지만 왜 @Lazy를 사용하면 아래와 같이 오류가 나는지 조사가 필요하다.  

{% asset_img "spring-circular-reference.png" %}  

추가적으로 @Configuration 대신 @EnableAutoCofiguration로 바꾸면 @Lazy를 사용하지 않아도 되는데 왜그런지 추가 조사를 해보겠다.