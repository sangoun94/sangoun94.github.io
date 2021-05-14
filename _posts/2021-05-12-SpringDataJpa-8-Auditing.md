---
layout: post
title: Spring Data Jpa
subtitle: Auditing
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpaExercise2/springdatajpa.jpg
cover-img: /assets/img/natural_design.jpg
tags: [spring data jpa, Auditing]
comments: true
categories: jpa
---

___
## 목표

#### Auditing 이 무엇이며 적용할 수 있다.

___

### Auditing 이란?

의심가는 데이터베이스의 작업을 모니터링 하고, 기록 정보를 수집 하는 기능이다.

<u>어느시간때</u>에 어떤 작업들이 주로 발생하는지, 어떤 작업을 <u>누가</u> 하는지 추적 할 수 있다.

> - 등록일
> - 수정일
> - 등록자
> - 수정자

위 네가지 컬럼들을 모든 테이블에 공통으로 넣어두고 관리를 하자는 취지이다.

[순수 JPA]

~~~java
@MappedSuperclass 
@Getter
public class JpaBaseEntity {
    @Column(updatable = false)
    private LocalDateTime createdDate;
    private LocalDateTime updatedDate;
    
    @PrePersist // manager persist 의해 처음 호출될 때
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        updatedDate = now;      // 초기화 안해주면 null 들어가서 신경쓸게 많아짐
    }
    
    @PreUpdate  //SQL UPDATE 이전
    public void preUpdate() {
        updatedDate = LocalDateTime.now();
    }
}
~~~

> - @MappedSuperclass 는 속성만 부여 Entity도 아니고 table도 아님
> - createdDate 는 생성 후 바뀌면 안되서 갱신 불가 설정 updatable = false
> - @PrePersist = 엔티티를 영속성컨텍스트에 관리하기 직전에 호출
> - @PreUpdate = 엔티티를 데이터베이스에 수정하기 직전에 호출

<br/>

위 JpaBaseEntity 적용

~~~java

@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id","username","age"})
public class Member extends JpaBaseEntity{

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

> - extends JpaBaseEntity 상속 끝

<br/>


~~~java
@Test
public void JpaEventJpaBaseEntity() throws Exception{
    //given
    Member member = new Member("member1", 10);
    memberRepository.save(member);
    Thread.sleep(100);
    member.setUsername("member2");  // 더티체킹
    em.flush();     //@PreUpdate
    em.clear();
    
    //when
    Member member1 = memberRepository.findById(member.getId()).get();
    
    //then
    System.out.println("member1 createdDate = " + member1.getCreatedDate());
    System.out.println("member1 updatedDate = " + member1.getLastModifiedDate());    
}
~~~


<strong>실행결과</strong>
~~~
member1 createdDate = 2021-05-12T16:50:47.148335
member1 updatedDate = 2021-05-12T16:50:47.339099
~~~

create, update 날짜시간 정보가 들어감.

<br/>

[DATA JPA]

~~~java
@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {

	public static void main(String[] args) {
		SpringApplication.run(DataJpaApplication.class, args);
	}
}
~~~

> - @EnableJpaAuditing 설정

<br/>

~~~java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;
    
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    private String lastModifiedBy;
}
~~~

> - @EntityListeners(AuditingEntityListener.class) 설정
> - @CreatedDate(등록일), @LastModifiedDate(수정일) 속성 설정
> - @CreatedBy(등록자), @LastModifiedBy(수정자) 속성 설정

<br/>

<strong>등록자, 수정자를 처리해주는 AuditorAware 스프링 빈 등록</strong>

LocalDateTime 은 자동 셋팅 가능

String 값 세팅 필요.

~~~java
@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {

	public static void main(String[] args) {
		SpringApplication.run(DataJpaApplication.class, args);
	}
	
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.of(UUID.randomUUID().toString());
    }
}
~~~

> - AuditorAware<String> 로 String 설정
> - 실무에선 SessionId, 시큐리티 정보 Id를 받아옴.
> - 예제에선 UUID로 진행

<br/>

~~~java
@Test
public void JpaEventJpaBaseEntity() throws Exception{
    //given
    Member member = new Member("member1", 10);
    memberRepository.save(member);
    Thread.sleep(100);      // 수정 결과 보기 위해 0.1초 지연
    member.setUsername("member2");  // 더티체킹
    em.flush();     //@PreUpdate
    em.clear();
    
    //when
    Member member1 = memberRepository.findById(member.getId()).get();
    
    //then
    System.out.println("member1 createdDate = " + member1.getCreatedDate());
    System.out.println("member1 updatedDate = " + member1.getLastModifiedDate());
    System.out.println("member1 CreateBy= " + member1.getCreateBy());
    System.out.println("member1 LastModifiedBy= " + member1.getLastModifiedBy());
}
~~~

<br/>

<strong>실행결과</strong>

~~~
member1 createdDate = 2021-05-12T17:02:19.267037
member1 updatedDate = 2021-05-12T17:02:19.446066
member1 CreateBy= fe487332-453e-4486-8efa-d1185ce1dbef
member1 LastModifiedBy= e3c9324a-d375-4af2-a771-2d49ef2939ff
~~~

모두 정상으로 나온다.

<br/>

<span style="color:pink">참고

~~~
memberRepository.save(member);
~~~

저장 시점에는 등록일, 등록자, 수정일, 수정자 같은 데이터가 저장된다.

이렇게 해두면 변경 컬럼만 확인해도 마지막에 업데이트한 유저를 확인 할 수 있으므로
유지보수 관점에서 편리하다.

~~~
@EnableJpaAuditing(modifyOnCreate = false)
~~~
저장시점에 원하는 데이터만 저장하고싶다면 위 코드 삽입한다.

~~~
member.setUsername("member2");
~~~

이 실행되면서 더티체킹이 일어나고 수정일과 수정자가 이 때 바뀌게 된다.

<br/>

### 날짜와 사람 분리

어느 테이블에는 수정자와 등록자를 추적할 필요가 없다. 대신 수정일과 등록일은 필요한 경우가 있을 것이다.

이럴 때 날짜와 사람을 분리해보려한다.

주로 날짜는 모든 테이블에서 사용하고 등록자,수정자는 필요한 테이블에만 추가해주는 형식을 갖는다.

~~~java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseTimeEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}



@Getter
public class BaseEntity extends BaseTimeEntity{

    @CreatedBy
    @Column(updatable = false)
    private String createBy;

    @LastModifiedBy
    private String lastModifiedBy;

}
~~~

> - 날짜만 필요하면 Entity 에 BaseTimeEntity 상속
> - 날짜 사람 다 필요하면 BaseEntity 상속 받아 사용