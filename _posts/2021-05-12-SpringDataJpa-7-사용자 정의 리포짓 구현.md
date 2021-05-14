---
layout: post
title: Spring Data Jpa
subtitle: 사용자 정의 repository
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpaExercise2/springdatajpa.jpg
cover-img: /assets/img/natural_design.jpg
tags: [spring data jpa, 사용자 정의 repository]
comments: true
categories: jpa
---

___
## 목표

#### customizing 할 수 있는 jpa를 사용할 수 있다.

___

여태까지 jpa 에서 자동으로 해주는 마법같은 기능들을 직접 코딩해보았다.

이번엔 JpaRepository 에서 구현해준 기능들을 쓰는게 아닌 detail 하게 내가 Customizing 할 수 있는 방법을 소개해보려한다.

만약 Jpa 제공하는 기능들을 직접 구현한다고 하면 당장 필요하지 않는 인터페이스들도 구현해야해서 비효율이다.

> - JPA 직접 사용( EntityManager )
> - 스프링 JDBC Template 사용
> - MyBatis 사용
> - 데이터베이스 커넥션 직접 사용 등등...
> - Querydsl 사용

등의 이유로 사용자 정의 repository 가 필요하다.

예제는 간단하게 코딩하고 하는 방법에 초점을 맞추겠다.

<br/>

<em>사용자 정의 repository</em>

~~~
public interface MemberRepository extends JpaRepository<Member, Long>
~~~

기존 repository 는 그대로 두고

새로운 interface 와 이놈의 impl 을 만들어준다.

~~~java
public interface MemberRepositoryCustom {
    List<Member> findMemberCustom();
}


@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom{

    private final EntityManager em;

    public List<Member> findMemberCustom() {
        return em.createQuery(
                "select m from Member m"
        ).getResultList();
    }
}
~~~

impl 에서 jpa query 를 customizing 해준다.

간단하게 Member 조회하는 쿼리로 진행하였다.

이제 이 사용자 정의 interface 를 기존 repository 에 추가해주면 끝난다.

~~~
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom
~~~

이렇게 추가해주면 findMemberCustom() 를 사용할 수 있게된다.

<span style="color:red">규칙

> - (MemberRepository, MemberRepositoryCustom) + Impl