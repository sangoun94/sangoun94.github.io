---
layout: post
title: Spring Data Jpa
subtitle: 반환 타입, 페이징 정렬
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpa.png
cover-img: /assets/img/natural_design.jpg
tags: [spring data jpa, 반환 타입, 페이징 정렬]
comments: true
categories: jpa
---

___
## 목표

#### 1. Spring data jpa 에서의 반환 타입들을 알 수 있다.
#### 2. Spring data jpa 에서의 페이징 처리를 알 수 있다.

___

스프링 데이터 JPA는 유연한 반환 타입 지원해준다.

<em>Return Type</em>

~~~java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsername(String name); //컬렉션
    Member findByUsername(String name); //단건
    Optional<Member> findByUsername(String name); //단건 Optional
    Page<Member> findByAge(int age, Pageable pageable);
}
~~~

<br/>

컬렉션 조회

> - 결과 없음 : size 0 반환

<br/>

단건 조회

> - 결과 없음: null 반환
> - 결과 2건 이상 : NoResultException 발생
> - 단건 조회는 jpa에서 getSingleResult() 메서드 호출
> - 여기서 결과가 없어도 NoResultException 뜨지만 jpa 가 null 처리

<br/>

<em>페이징 처리</em>


[순수 jpa 페이징 처리](https://sangoun94.github.io/2021-05-06-jpa-%ED%99%9C%EC%9A%A92-3-API%EA%B0%9C%EB%B0%9C%EA%B3%A0%EA%B8%899-%EC%BB%AC%EB%A0%89%EC%85%98-%EC%A1%B0%ED%9A%8C-%EC%B5%9C%EC%A0%81%ED%99%944/)

이번엔 data jpa 로 페이징 처리를 해보겠다.


~~~java
public interface MemberRepository extends JpaRepository<Member, Long> {
    Page<Member> findByAge(int age, Pageable pageable);
}
~~~

"findByAge(int age, Pageable pageable);"를 추가 후 test 코드를 작성하겠다.

~~~java
@SpringBootTest
@Transactional
//@Rollback(value = false)
public class MemberRepositoryTest {
    @Test
    public void paging() {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        int age = 10;
        PageRequest pag = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

        //when
        Page<Member> page = memberRepository.findByAge(age, pag);

        Page<MemberDto> map = page.map(m -> new MemberDto(m.getId(), m.getUsername(), null));


        //then
        List<Member> content = page.getContent();
        for (Member member : content) {
            System.out.println("member = " + member);
        }
        long totalElements = page.getTotalElements();
        System.out.println("totalElements = " + totalElements);


        assertThat(content.size()).isEqualTo(3);
        assertThat(page.getTotalElements()).isEqualTo(5);
        assertThat(page.getNumber()).isEqualTo(0);
        assertThat(page.getTotalPages()).isEqualTo(2);
        assertThat(page.isFirst()).isTrue();
        assertThat(page.hasNext()).isTrue();
    }
}
~~~

> - member 생성
> - PageRequest.of 로 페이지 설정들을 넣어준다. (page,size, sort, properties) 순서, Page는 1부터 시작이 아니라 0부터 시작이다.
> - Paging 처리 후 dto 변환
> - Paging 에선 content(내용), totalElement(총 수), Pageable 내용을 가져옴 

paging 처리하여 전체 조회된 것에 대해 count 쿼리를 하게 된다면 굉장히 무거운 결과가 나오게 된다. 조회건이 10만개라고 하자.

조회 쿼리가 실행될 때 페이징 처리가 된다.

페이징 처리가 되어 count 되어진다. 10만건에 대해 count를 치기 때문에 data Query 와 count Query 를 분리 할 필요가 있다.

~~~java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query(value = “select m from Member m”,
            countQuery = “select count(m.username)from Member m”)
    Page<Member> findMemberAllCountBy(Pageable pageable);
}
~~~

순수 jpa 와 spring data jpa 에서 처리하는 Paging 차이를 알고 data jpa 가 많은 부분을 도와준다는 사실을 알아야한다.