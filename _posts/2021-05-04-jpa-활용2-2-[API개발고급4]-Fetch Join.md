---
layout: post
title: JPA 활용[2] 성능개선 - API개발고급
subtitle: Fetch Join
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpa.png
cover-img: /assets/img/natural_design.jpg
tags: [jpa, API고급, Fetch Join, DTO]
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

__패치 조인 최적화__

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


    @GetMapping("/api/v3/simple-orders")
    public List<SimpleOrderDto> ordersV3() {
        List<Orders> orders = orderRepositoy.findAllWithMemberDelivery();
        List<SimpleOrderDto> returnResult = orders.stream()
                .map(SimpleOrderDto::new)
                .collect(toList());
        return returnResult;
    }

    @Data
    static class SimpleOrderDto{
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;

        public SimpleOrderDto(Orders order) {
            orderId = order.getId();
            name = order.getMember().getName();     //Lazy 초기화 시점.
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();     //Lazy 초기화 시점.
        }
    }
}
~~~

dto로 변환하는 부분은 똑같다. findAllWithMemberDelivery()를 확인해보자.

~~~java
@Repository
@RequiredArgsConstructor
public class OrderRepositoy {
    private final EntityManager em;

    // fetch 조인
    public List<Orders> findAllWithMemberDelivery() {
        return em.createQuery(
                "select o from Orders o " +
                        "join fetch o.member m " +
                        "join fetch o.delivery d", Orders.class)
                .getResultList();

    }
}
~~~

> - Repository 작성하였다.
> - List<Orders> 엔티티 리스트를 반환한다.
> - Query를 작성하였고 "join fetch" 사용하여 작성하였다.
> - Orders.class로 반환한다.
> - getResultList()는 결과에 대한 List를 반환한다.

이렇게 fetch 조인을 하였을 때 과연 N + 1 문제가 해결이 될까?? 실행을 해보자.

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

전에 결과랑 같은 결과를 반환해준다.

~~~sql
select
    orders0_.order_id as order_id1_6_0_,
    member1_.member_id as member_i1_4_1_,
    delivery2_.delivery_id as delivery1_2_2_,
    orders0_.delivery_id as delivery4_6_0_,
    orders0_.member_id as member_i5_6_0_,
    orders0_.order_date as order_da2_6_0_,
    orders0_.status as status3_6_0_,
    member1_.city as city2_4_1_,
    member1_.street as street3_4_1_,
    member1_.zipcode as zipcode4_4_1_,
    member1_.name as name5_4_1_,
    delivery2_.city as city2_2_2_,
    delivery2_.street as street3_2_2_,
    delivery2_.zipcode as zipcode4_2_2_,
    delivery2_.status as status5_2_2_ 
from
    orders orders0_ 
inner join
    member member1_ 
        on orders0_.member_id=member1_.member_id 
inner join
    delivery delivery2_ 
        on orders0_.delivery_id=delivery2_.delivery_id
~~~

Query가 하나만 나온다. 이게 무슨 일인가???

join fetch는 보통의 sql에서 지원하지 않는 명령어이다. 

JPA에서 N + 1 문제를 해결하기 위한 방안으로 join fetch 를 지원해주었고 query를 보다시피 member와 delivery를 전부 select하여 전체 조회를 하였다.

그럼 또 하나의 궁금증이 생긴다.

내가 원하는 api의 결과에는 member, delivery가 필요하지 않는데 좀 더 최적화를 해보고 싶다!

다음 포스터에선 이것보다 한 단계 더 최적화된 내용으로 기재하겠다.