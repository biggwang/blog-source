---
title: Spock 으로 Controller 파일 업로드 테스트 해보기
catalog: true
date: 2019-07-25 20:05:15
subtitle: 
header-img: "bg_computer.jpg"
tags: 
- Experience
catagories:
- Spock
---

# 들어가며
Junit 으로만 사용하다가 BDD에 특화된 테스트 프레임워크인 Spock을 팀내 테스트 코드에 공식 툴?로 정하게 되어서 Spock에 얼른 익숙해지기 위해 경험했던 내용을 공유합니다.


# 도전
Controller에서 파일 업로드 테스트를 합니다.  

테스트 케이스는  
- 자기소개 텍스트 길이는 10자 이내여야 합니다.
- 자기소개 이미지 있을때 없을때 경우 테스트 합니다.


우선 전체 코드를 한번 보고 부분부분 설명해 드리겠습니다. 

~~~ java
@SpringBootTest(classes = FrontWebApplication.class)
@AutoConfigureMockMvc
@Slf4j
@ActiveProfiles("local")
class MemberApiControllerV1Test extends Specification {

    
    @Autowired
    private MockMvc mockMvc;

    @Unroll
    def "자기소개 이미지 파일 없을 경우 자기소개 업데이트만 수행한다."() {

        when:
        MockMultipartFile file = new MockMultipartFile("test.json", "", "application/json", "{\"key1\": \"value1\"}".getBytes());

        MockMultipartHttpServletRequestBuilder builder = MockMvcRequestBuilders.multipart("/api/member/v1/sign")
                .with(new RequestPostProcessor() {
                    @Override
                    public MockHttpServletRequest postProcessRequest(MockHttpServletRequest request) {
                        request.setMethod("PUT")
                        return request
                    }
                })
                .header('User-Agent', TokenCheckAopTest.userAgent)
                .contentType(MediaType.APPLICATION_JSON)
                .cookie(new Cookie("쿠키명", "쿠키값"))

        MockHttpServletResponse respones = mockMvc.perform(builder
                .file(file))
                .andReturn().response

        then:
        println("### response content: " + respones.getContentAsString())
        respones.getStatus() == HttpStatus.SC_OK
    }
}
~~~



# Multipart를 PUT 메소드로 보낼 경우

기본적으로 MockMvcRequestBuilders 객체는 POST 만 지원되는 것으로 알고 있습니다.  
개발 코드에서는 @PutMapping으로 받고 있는데요 아무처리 없이 테스트를 하다 보면 403 지원하지 않는 Http Method 에러를 보시게 될 겁니다.  그것을 방지 하기 위한 코드가 아래에 있습니다.

~~~ java
MockMvcRequestBuilders.multipart("/api/member/v1/sign")
                .with(new RequestPostProcessor() {
                    @Override
                    public MockHttpServletRequest postProcessRequest(MockHttpServletRequest request) {
                        request.setMethod("PUT")
                        return request
                    }
                })
~~~


# Where 구문으로 여러 케이스 테스트해 보기

Spock을 가지고 테스트 코드를 작성하면서 가장 좋았던 장점이었던거 같습니다.  
Where 구문에 내가 테스트하고 싶은 케이스를 적고 케이스를 쭈욱~~ 돌리는 거죠 마치 for문이 돌면서 로직을 처리하는 것처럼 

**TODO: where 구문 들어간 코드 넣기**