---
layout: post
title: Spring Data Jpa
subtitle: 벌크성 수정 및 EntityGraph
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpa.png
cover-img: /assets/img/natural_design.jpg
tags: [spring data jpa, 벌크성 수정, EntityGraph]
comments: true
categories: jpa
---

___
## 목표

#### 1. 벌크성 수정이 무엇인지 알 수 있다.
#### 2. EntityGraph 알 수 있다.

___

<em>벌크성 수정</em>

### 벌크성 수정이란?

대량의 데이터들을 쿼리를 통해 수정한다는 의미. 대량 처리를 했을 시에는 속도가 빠르지만 한 행 처리에 있어서는 다소 느린 점이 있음.

[순수 JPA]

executeUpdate() 를 사용한다.

~~~java
public int bulkAgePlus(int age) {
    int resultCount = em.createQuery(
        "update Member m set m.age = m.age + 1" +
        "where m.age >= :age")
        .setParameter("age", age)
        .executeUpdate();
    return resultCount;
}
~~~

<br/>

[Data JPA]

@Modifying 가 executeUpdate() 역할

~~~java
@Modifying
@Query("update Member m set m.age = m.age + 1 whre m.age >= :age")
int bulkAgePlus(@Param("age") int age);

~~~

이렇게 설정해주고 test 해보자.

<br/>

~~~java
@Test
public void bulkUpdate() throws Exception {
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 20));
        memberRepository.save(new Member("member3", 30));
        memberRepository.save(new Member("member4", 40));
        memberRepository.save(new Member("member5", 50));
        memberRepository.save(new Member("member6", 60));

        int resultCount = memberRepository.bulkAgePlus(20);


        List<Member> result1 = memberRepository.findListByUsername("member6");
        Member member5_1 = result1.get(0);
        System.out.println("member5 = " + member5_1);      
}
~~~

<strong>실행결과</strong>

~~~sql
update member set age=age+1 where age>=20;
select member0_.member_id as member_i1_1_, member0_.created_date as created_2_1_, member0_.last_modified_date as last_mod3_1_, member0_.age as age4_1_, member0_.team_id as team_id6_1_, member0_.username as username5_1_ from member member0_ where member0_.username='member6';
~~~

정상적으로 SQL 뿌려진다.

member5_1 을 확인해보자.

~~~
member5_1 = Member(id=6, username=member6, age=60)
~~~

<span style="color:pink">이상하다. 다시 p.c.에서 찾은 result1의 age 결과가 60이다.. 쿼리는 "age + 1" 을 실행시켜 줬는데 반영이 안됐다.

db를 확인해보자.

![벌크성 수정 1](/assets/img/jpaExercise2/spring%20data%20jpa%20벌크성%20수정%201.png)

<span style="color:pink">DB 정상적으로 반영되어있다.

___

이번엔 p.c.를 초기화한 코드를 추가하고 실행해보겠다.



~~~java
@Test
public void bulkUpdate() throws Exception {
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 20));
        memberRepository.save(new Member("member3", 30));
        memberRepository.save(new Member("member4", 40));
        memberRepository.save(new Member("member5", 50));
        memberRepository.save(new Member("member6", 60));

        int resultCount = memberRepository.bulkAgePlus(20);


        List<Member> result1 = memberRepository.findListByUsername("member6");
        Member member5_1 = result1.get(0);
        System.out.println("member5 = " + member5_1);      
        
        em.flush();
        em.clear();

        List<Member> result2 = memberRepository.findListByUsername("member6");
        Member member5_2 = result2.get(0);
        System.out.println("member5_2 = " + member5_2);


        assertThat(resultCount).isEqualTo(5);
}
~~~


<strong>실행결과</strong>

~~~sql
update member set age=age+1 where age>=20;
select member0_.member_id as member_i1_1_, member0_.created_date as created_2_1_, member0_.last_modified_date as last_mod3_1_, member0_.age as age4_1_, member0_.team_id as team_id6_1_, member0_.username as username5_1_ from member member0_ where member0_.username='member6';
~~~

동일하다

member5_1과 member5_2를 확인해보자.

~~~
member5_1 = Member(id=6, username=member6, age=60)
member5_2 = Member(id=6, username=member6, age=61)
~~~

디비도 확인하자.

![벌크성 수정 2](/assets/img/jpaExercise2/spring%20data%20jpa%20벌크성%20수정%201.png)


이렇게 나오는 이유가 무엇인가..

알고보니 벌크성 수정은 영속성 컨텍스트를 무시하고 바로 db 반영을 시키는 놈이다.

고로:: 벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에, 영속성 컨텍스트에 있는 엔티티의 상태와
DB에 엔티티 상태가 달라질 수 있다.

<span style="color:pink">해결방안

벌크 연산 직후 영속성 컨텍스트를 초기화 시키는 것이다.

초기화 방법 

> - TEST 같이 em.flush; em.clear; 사용
> - @Modifying(clearAutomatically = true)  설정.

<br/>

<em>EntityGraph</em>

EntityGraph = 연관된 엔티티들을 SQL 한번에 조회하는 방법 (간소화된 Fetch Join)

참고
[fetch join 바로가기](https://sangoun94.github.io/2021-05-04-jpa-%ED%99%9C%EC%9A%A92-2-API%EA%B0%9C%EB%B0%9C%EA%B3%A0%EA%B8%894-Fetch-Join/)

<br/>

~~~java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Override
    @EntityGraph(attributePaths = {"team"})
    List<Member> findAll();

    //JPQL + 엔티티 그래프
    @EntityGraph(attributePaths = {"team"})
    @Query("select m from Member m")
    List<Member> findMemberEntityGraph();

    //메서드 이름으로 쿼리에서 특히 편리하다.
    @EntityGraph(attributePaths = {"team"})
    List<Member> findByUsername(String username)
}
~~~

data jpa 에서 손쉽게 Query 에서 fetch join을 하는 것이 아닌 @EntityGraph 로 fetch join 가능하다.

@EntityGraph 는 left outer join 을 사용하게 된다.

jpql 과 같이 사용 할 수 있고 쿼리 메서드로도 가능하다.






