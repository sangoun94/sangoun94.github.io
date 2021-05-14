---
layout: post
title: Spring Data Jpa
subtitle: 예제 도메인 모델
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpaExercise2/springdatajpa.jpg
cover-img: /assets/img/natural_design.jpg
tags: [도메인 모델, spring data jpa]
comments: true
categories: jpa
---

___
## 목표

#### 예제 도메인 모델 확인.
___

<br/>

앞서 배운 순수 JPA를 사용하여 Web Application과 성능최적화를 해보았다. 실무에서 주로 사용하는 Spring Data Jpa로 구현해보겠다.

사용하는 기술

> - springboot
> - H2
> - Spring Data Jpa
> - 화면은 없으며 test코드를 사용함.

<br/>

설정에 관한 내용은 아래 링크를 참고하면 된다.

### [H2-JPA 설정](https://sangoun94.github.io/2021-04-19-h2-jpa/)

<br/>

먼저 엔티티, erd 관계 부터 간단하게 짚고 넘어가자

![spring data jpa 예제 도메인](/assets/img/jpaExercise2/2021-05-11-SpringDataJpa-1-예제%20도메인.md)


아주 간단한 예제이다. 도메인을 만들어보자.

<br/>

<em>Member Entity</em>

~~~java
@Entity
@Getter
@Setter //안쓰는게 정신건강에 이롭다. 예제니깐 쓴다
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id","username","age"})
public class Member extends BaseTimeEntity{

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;
    private String username;
    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;


    public Member(String username, int age){
        this.username = username;
        this.age = age;
    }

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        if (team != null) {
            changeTeam(team);
        }
    }

    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }

}
~~~

<br/>

> - 기존에 배운 공부들로 이해할 수 있다. team에 대해 @ManyToOne 양방향 맵핑을 하였고 "team"이 주인이 된다.
> - @xToOne 은 FetchType에 default 가 Eager여서 LAZY로 바꿔준다
> - 기본생성자를 public이 아닌 Protected로 해준다. => 무분별한 생성자 제한.

<br/>

~~~java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@ToString(of = {"id", "name"})
public class Team extends BaseTimeEntity{

    @Id
    @GeneratedValue
    @Column(name = "team_id")
    private Long id;
    private String name;


    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    public Team(String name) {
        this.name = name;
    }
}
~~~

<br/>

> - 양방향 맵핑이기에 mappedBy로 team(주인) 섬긴다.



아주 간단한 예제로 spring-data-jpa를 다뤄볼려고한다. 다음 시간엔 해당 엔티티에 대한 공통 인터페이스를 만들겠다.