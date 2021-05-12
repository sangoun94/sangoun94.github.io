---
layout: post
title: Spring Data Jpa
subtitle: 쿼리 정의 및 param 바인딩
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpa.png
cover-img: /assets/img/natural_design.jpg
tags: [쿼리 정의, spring data jpa]
comments: true
categories: jpa
---

___
## 목표

#### 쿼리 정의를 해본다.
___

저번 포스터에서 쿼리 메소드를 사용하여 따로 Query를 짜지 않고 손쉽게 쿼리를 생성하고 사용할 수 있었다.

이번 시간에는 좀 더 복잡하고 쿼리 메소드 만으로는 해결하기 어려운 부분들을 해결해야만 한다. 직접 쿼리를 정의해보겠다.

<em>Query Definition</em>


~~~java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("select m from Member m where m.username= :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);
}
~~~

<br/>

끝이다..

spring data jpa 에서 제공하는 @Query 에 직접 

이 메소드의 내용은 이렇게 표현할 수 있다.

> - 쿼리 메서드만으론 코드가 너무 지저분.
> - 애플리케이션 실행 시점에 문법 오류를 발견.
> - 간단한 쿼리들은 @Query 자주 사용
> - :age 와 같이 이름기반 파라미터 바인딩 사용 (위치기반 파라미터 x)
> - @Param 으로 바인딩.

<br/>

Collection 도 @Param 으로 바인딩 가능하다.

~~~java
@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") List<String> names);
~~~





