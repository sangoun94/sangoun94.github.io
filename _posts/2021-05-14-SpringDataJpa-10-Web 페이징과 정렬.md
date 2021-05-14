---
layout: post
title: Spring Data Jpa
subtitle: Web 페이징과 정렬
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpaExercise2/springdatajpa.jpg
cover-img: /assets/img/natural_design.jpg
tags: [spring data jpa, web, 페이징과 정렬]
comments: true
categories: jpa
---

___
## 목표

#### MVC에서 페이징과 정렬을 할 수 있다.

___

간단한 예제를 통해 알아보겠다.

~~~java
@ResponseBody
@GetMapping("/helloPage")
public Page<Member> helloPage(Pageable pageable) {

        pageable = PageRequest.of(10, 10);
    
        Member member = new Member("sangoun", 28);
        memberRepository.save(member);

        Page<Member> pageAll = memberRepository.findAll(pageable);
        return pageAll;
        }

}
~~~

~~~java
    @Test
public void webPage() throws Exception{
        mockMvc.perform(get("/helloPage"))
        .andDo(print())
        .andExpect(status().isOk());
}

~~~

<strong> 실행결과</strong>

~~~json
{
  "content": [
    {
      "createdDate": "2021-05-14T11:08:00.3162804",
      "lastModifiedDate": "2021-05-14T11:08:00.3162804",
      "createdBy": "4948797d-fc2e-46a2-ab38-641ec9878ddc",
      "lastModifiedBy": "4948797d-fc2e-46a2-ab38-641ec9878ddc",
      "id": 1,
      "username": "sangoun",
      "age": 28,
      "team": null
    }
  ],
  "pageable": {
    "sort": {
      "sorted": false,
      "unsorted": true,
      "empty": true
    },
    "pageNumber": 0,
    "pageSize": 10,
    "offset": 0,
    "unpaged": false,
    "paged": true
  },
  "last": true,
  "totalPages": 1,
  "totalElements": 1,
  "number": 0,
  "sort": {
    "sorted": false,
    "unsorted": true,
    "empty": true
  },
  "first": true,
  "numberOfElements": 1,
  "size": 10,
  "empty": false
}
~~~

content, pageable 등 페이징 처리가 된걸 알 수 있다.

PageRequest 를 통해 Pageable 커스텀 가능하다.

> - page: 현재 페이지, 0부터 시작한다.
> - size: 한 페이지에 노출할 데이터 건수
> - sort: 정렬 조건을 정의한다. 예) 정렬 속성,정렬 속성...(ASC | DESC), 정렬 방향을 변경하고 싶으면 sort 파라미터 추가 ( asc 생략 가능)

<br/> 

PageRequest 사용해서 소스 안에 정의할 수 있지만, 초기화 단계에서 정의도 가능하다.

@PageableDefault 사용해서 커스텀 해보자.

~~~java

@ResponseBody
@GetMapping("/helloPage")
public Page<Member> helloPage(@PageableDefault(size = 12, sort = "username", direction = Sort.Direction.DESC) Pageable pageable) {
    Member member = new Member("sangoun", 28);
    memberRepository.save(member);
    Page<Member> pageAll = memberRepository.findAll(pageable);
    return pageAll;
}
~~~

<br/>

 size, page 글로벌 설정을 application.yml 설정 페이지에서 가능하다.


 
~~~yml
spring:
  data:
    web:
      pageable:
        default-page-size: 10
        max-page-size: 2000
~~~