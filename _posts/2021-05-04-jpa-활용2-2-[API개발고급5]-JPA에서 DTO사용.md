---
layout: post
title: JPA 활용[2] 성능개선 - API개발고급
subtitle: JPA에서 DTO사용
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpa.png
cover-img: /assets/img/natural_design.jpg
tags: [jpa, API고급, DTO]
comments: true
categories: jpa
---

___
## 목표

#### 1. 조회용 샘플 데이터 입력 [준비]
#### 2. 지연 로딩과 조회 성능 최적화 할 수 있다.
#### 3. 컬렉션 조회 최적화 하는 방법을 알 수 있다.
#### 4. 페이징과 한계 돌파를 할 수 있다.
#### 5. OSIV와 성능 최적화에 대해 알아보자.
___

<br/>

준비된 db 값들로 지연로딩에서의 조회 성능 최적화 단계별 진행 한다.

| 단계 |
|:---:|
| `엔티티 노출` |
| `DTO 변환` |
| `패치 조인 최적화` |
| `DTO 바로 조회` |

___

__DTO 바로 조회__

~~~java
/**
 * XtoOne 관계 (ManyToOne, OneToOne)
 * Order 조회
 * Order -> Member 연관   ManyToOne
 * Order -> Delivery 연관 OneToOne
 */
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
    private final OrderRepositoy orderRepositoy;

    @GetMapping("/api/v4/simple-orders")
    public List<OrderSimpleQueryDto> ordersV4() {
        return orderRepositoy.findOrderDto();
    }
}
~~~

이번엔 dto 클래스를 새로 만들 것이다. 하지만 형식은 같다.

~~~java
@Data
public class OrderSimpleQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public OrderSimpleQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;     //Lazy 초기화 시점.
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;     //Lazy 초기화 시점.
    }
}
~~~

전에 사용하던 controller안에서의 dto와 다른 건 생성자에 orders를 넣은게 아닌 각 필드마다 매칭시켜준 것 뿐이다.

이제 findOrderDto를 보자.

~~~java
@Repository
@RequiredArgsConstructor
public class OrderRepositoy {
    private final EntityManager em;

    public List<OrderSimpleQueryDto> findOrderDto() {
        return em.createQuery(
                "select new com.jpabook.jpashop.repository.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address) " +
                        "from Orders o " +
                        "join o.member m " +
                        "join o.delivery d", OrderSimpleQueryDto.class).
                getResultList();
    }
}
~~~

> - Repository에 작성
> - List<OrderSimpleQueryDto> 반환,, List<Orders>가 아님
> - select문에서도 dto 생성자를 가져온다.

일단 실행해보자.

~~~json
[
    {
        "orderId": 4,
        "name": "userA",
        "orderDate": "2021-05-04T13:37:02.650656",
        "orderStatus": "ORDER",
        "address": {
            "city": "서울",
            "street": "1",
            "zipcode": "1111"
        }
    },
    {
        "orderId": 11,
        "name": "userB",
        "orderDate": "2021-05-04T13:37:02.717656",
        "orderStatus": "ORDER",
        "address": {
            "city": "전주",
            "street": "2",
            "zipcode": "2222"
        }
    }
]
~~~

실행 결과는 동일하게 출력되었다.

다음 n+1 문제는 어떻게 되었는지 보자.

~~~sql
select
    orders0_.order_id as col_0_0_,
    member1_.name as col_1_0_,
    orders0_.order_date as col_2_0_,
    orders0_.status as col_3_0_,
    delivery2_.city as col_4_0_,
    delivery2_.street as col_4_1_,
    delivery2_.zipcode as col_4_2_ 
from
    orders orders0_ 
inner join
    member member1_ 
        on orders0_.member_id=member1_.member_id 
inner join
    delivery delivery2_ 
        on orders0_.delivery_id=delivery2_.delivery_id
~~~

> - N + 1 문제도 해결되면서 member, delivery의 정보도 잘 빠져서 나옴.
> - SQL 사용하는 것처럼 원하는 값을 선택해서 조회
> - new 명령어 사용, JPQL의 결과가 DTO로 반환
> - select절에 원하는 데이터 조회 가능, 네트웍 용량 최적화 유리(미비)
> - 재사용 떨어짐, API 스펙 바뀌면 못쓰는 코드이다.

근데 이 DTO를 사용하는 방법이 제일 BEST라고 말할 수 있을까? 또, 언제 사용하는게 좋을까?

많은 사용들이 사용하는 API여서 트래픽이 정말 심할 때 사용하는 것이 옳다. 또한 select 절에서 원하는 컬럼들이 50,100개 이렇게 많아질 때 사용하는 게 좋다.

<br/>

###### 마지막으로 쿼리 방식 권장 순서 정리하고 마치겠다.

> - 쿼리 방식 선택 권장 순서
> - 1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.
> - 2. 필요하면 페치 조인으로 성능을 최적화 한다. 대부분의 성능 이슈가 해결된다.
> - 3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.
> - 4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접

