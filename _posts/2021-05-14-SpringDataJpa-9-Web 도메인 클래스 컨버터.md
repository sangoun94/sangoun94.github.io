---
layout: post
title: Spring Data Jpa
subtitle: Web 도메인 클래스 컨버터
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpaExercise2/springdatajpa.jpg
cover-img: /assets/img/natural_design.jpg
tags: [spring data jpa, web, 도메인 클래스 컨버터]
comments: true
categories: jpa
---

___
## 목표

#### 도메인 클래스 컨버터 이 무엇이며 활용할 수 있다.

___



~~~java
@Controller
@RequiredArgsConstructor
public class HelloController {

    private final MemberRepository memberRepository;

    @ResponseBody
    @GetMapping("/helloDomain/{id}")
    public String domainConvert(@PathVariable("id") Long id) {
        Optional<Member> byIdid = memberRepository.findById(id);
        Member member = byIdid.get();
        return member.getUsername();
    }

}
~~~

아주 간단하게 url에 id 받아서 username 결과 뽑는 코드이다. 

간단하게 test해보자 mock 처음쏴봐서 공부하면서 진행하고있다. (test 코드는 따로 카테고리 따서 공부하면서 포스팅하겠다.)

~~~java
@SpringBootTest
@AutoConfigureMockMvc
class HelloControllerTest {

    @Autowired
    MemberRepository memberRepository;
    @Autowired
    MockMvc mockMvc;

    public void fsk() throws Exception{
        //given
        Member member = new Member("sangoun", 28);
        memberRepository.save(member);
        //when
        mockMvc.perform(get("/helloDomain/" + member.getId()))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("jpa"));
        //then
    }
}
~~~

<strong> 실행결과</strong>

~~~
java.lang.AssertionError: Response content expected:<jpa> but was:<sangoun>
Expected :jpa
Actual   :sangoun
~~~

일부러 실했지만 결과가 아주 실하게 잘 나왔다.

도메인 클래스 컨버터는 Spring Data Jpa를 사용하게 되면 Common 프로젝트에 있는 클래스 인데, 간단하게, id-> 엔티티 타입으로 변환 해주거나, 엔티티 -> id 로 변환해 주는 컨버터를 제공해준다. 예제를 통해서 동작하는 지 살펴보자.

<br/>

<em>도메인 클래스 컨버터</em>

이제 도메인 클래스 컨버터를 살펴보자.

~~~java
public class DomainClassConverter<T extends ConversionService & ConverterRegistry>
        implements ConditionalGenericConverter, ApplicationContextAware {

    private final T conversionService;
    private Repositories repositories = Repositories.NONE;
    private Optional<ToEntityConverter> toEntityConverter = Optional.empty();
    private Optional<ToIdConverter> toIdConverter = Optional.empty();
...

~~~

ToEntityConverter와 ToIdConverter 클래스가 위에 컨트롤러에서 작성했던 역활과 동일한 일을 한다. 

때문에 아까전의 컨트롤러 코드에서 다음과 같이 바꿀 수 있다.

<br/>

~~~java
@Controller
@RequiredArgsConstructor
public class HelloController {

    private final MemberRepository memberRepository;

    @ResponseBody
    @GetMapping("/helloDomain/{id}")
    public String domainConvert(@PathVariable("id") Member member) {
        return member.getUsername();
    }

}
~~~

@PathVariable("id") id를 바로 엔티티에서 조회 후 해당 엔티티를 가져온다

테스트 해보겠다.

<br/>

~~~
java.lang.AssertionError: Response content expected:<jpa> but was:<sangoun>
Expected :jpa
Actual   :sangoun
~~~

동일하게 에러가 뜨고 정상적으로 데이터 출력한다.

<br/>

<span style="color: red">주의

도메인 클래스 컨버터로 엔티티를 파라미터로 받으면, 이 엔티티는 단순 조회용으로만 사용해야 한다.

(트랜잭션이 없는 범위에서 엔티티를 조회했으므로, 엔티티를 변경해도 DB에 반영되지 않는다.)