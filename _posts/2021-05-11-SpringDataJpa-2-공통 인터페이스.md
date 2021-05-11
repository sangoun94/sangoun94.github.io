---
layout: post
title: Spring Data Jpa
subtitle: 공통 인터페이스
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpa.png
cover-img: /assets/img/natural_design.jpg
tags: [CRUD, spring data jpa]
comments: true
categories: jpa
---

___
## 목표

#### Member, Team CRUD
___



~~~java
public interface MemberRepository extends JpaRepository<Member, Long> {
    
}
~~~

<br/>



끝이다..

JpaRepository<T, id> => 사용할 entity 타입과 id를 넣어주면 된다.

기본적인 CRUD 구현이 끝났다. 써먹어보자.

<br/>

~~~java
@SpringBootTest
@Transactional
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @PersistenceContext
    EntityManager em;

    @Test
    public void testMember() {
        Member member = new Member("MemberA", 10);
        Member saveMember = memberRepository.save(member);

        Member findMember = memberRepository.findById(saveMember.getId()).get();

        assertThat(findMember.getId()).isEqualTo(member.getId());
//      assertThat(findMember.getName()).isEqualTo(member.getName());
        assertThat(findMember).isEqualTo(member);
    }
    
    @Test
    public void basicCRUD() {
        Member member1 = new Member("member1");
        Member member2 = new Member("member2");
        memberRepository.save(member1);
        memberRepository.save(member2);
        
        //단건 조회 검증
        Member findMember1 = memberRepository.findById(member1.getId()).get();
        Member findMember2 = memberRepository.findById(member2.getId()).get();
        assertThat(findMember1).isEqualTo(member1);
        assertThat(findMember2).isEqualTo(member2);
        
        //리스트 조회 검증
        List<Member> all = memberRepository.findAll();
        assertThat(all.size()).isEqualTo(2);
        
        //카운트 검증
        long count = memberRepository.count();
        assertThat(count).isEqualTo(2);
        
        //삭제 검증
        memberRepository.delete(member1);
        memberRepository.delete(member2);
        long deletedCount = memberRepository.count();
        assertThat(deletedCount).isEqualTo(0);
    }
}
~~~

<br/>


> - testMember 는 기존 entity가 잘 들어가는지 확인
> - basicCRUD 에서 CRUD 테스트 확인

<br/>

이상하다 나는 아무런 구현을 해주지 않았는데 CRUD가 돌아갔다.. 순수 jpa 라면 em.persist, delete 등 구현해주어야했던 것들이 spring data jpa 가 해주는 것이다.

초고수 프로그래머 분이 공통으로 사용되는 CRUD와 같은 기능들을 JpaRepository 만 상속받으면 사용할 수 있게끔 해줌.

실제로 JpaRepository 의 구현제는 SimpleJpaRepository 에서 구현해준다.


<span style="color:red">참고</span>

~~~java
@EnableJpaRepositories(basePackages = "페키지 경로")
~~~

JpaRepository 를 사용하기 위해선 위와같은 설정이 필요하지만 @SpringBootApplication이 위치를 지정해서 SpringBoot를 사용하면 위의 설정은 생략 가능하다.