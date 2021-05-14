---
layout: post
title: Spring Data Jpa
subtitle: 쿼리 메소드
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpaExercise2/springdatajpa.jpg
cover-img: /assets/img/natural_design.jpg
tags: [Query Method, spring data jpa]
comments: true
categories: jpa
---

___
## 목표

#### 쿼리 메소드가 무엇인지 알 수 있다.
___

저번 포스터에서 마법같이 CRUD 가 자동으로 된 것을 알 수 있다. 이번엔 더 신기한 JPA의 기능들을 알아보려한다.

<em>Query Method</em>

전 포스터에서는 JpaRepository 안에 구성된 interface 기능들로 CRUD 를 사용하였다. 이번엔 where 절을 넣어서 원하는 쿼리를 만들어보자.

~~~java
public interface MemberRepository extends JpaRepository<Member, Long> {
    public List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
~~~

<br/>



끝이다..

이 메소드의 내용은 이렇게 표현할 수 있다.

~~~java
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
    return em.createQuery(
        "select m from Member m where m.username = :username and m.age > :age")
        .setParameter("username", username)
        .setParameter("age", age)
        .getResultList();
}
~~~

spring data jpa 에선 findByUsernameAndAgeGreaterThan() 의 구현은 필요없다.

### 왜 그런걸까?

spring data jpa 에선 메소드 이름을 분석해서 JPQL 을 생성 및 실행해준다.

find...By조건문들 => ...에는 아무거나 들어가도 무방하다.

자세한 규칙과 방법은 여길 들어가자.

[스프링 데이터 JPA 공식 문서 참고](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)

<span style="color:red">주의</span>

> - 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 변경
> - 그렇지 않으면 애플리케이션을 시작하는 시점에 오류 발생.
> - 애플리케이션 로딩 시점에 오류를 인지할 수 있는 것이 스프링 데이터 JPA의 매우 큰 장점.