---
layout: post
title: JPA 활용[1] WebApplication - 도메인 분석 설계
subtitle: 요구사항 및 도메인 모델과 테이블 설계.
gh-repo: daattali/beautiful-jekyll
thumbnail-img: /assets/img/jpa.png
cover-img: /assets/img/natural_design.jpg
tags: [jpa, API기본, 도메인 모델]
comments: true
categories: jpa
---

___
## 목표

#### 1. JPA Web_Application을 만들기 위한 요구사항 분석을 한다.
#### 2. 도메인 모델과 테이블 설계를 한다.
___

| 단계 |
|:---:|
| `요구 기능` |
| `요구사항 분석` |
| `도메인 설계` |
| `엔티티 설계` |
| `테이블 설계` |

___

<strong>사용 기술 : 

1. H2
2. SpringBoot
3. JPA
4. JUNIT5
5. 타임리프</strong>

<br/>


__요구 기능__

> - 회원 기능
> - 회원 등록
> - 회원 조회
> - 상품 기능
> - 상품 등록
> - 상품 수정
> - 상품 조회
> - 주문 기능
> - 상품 주문
> - 주문 내역 조회
> - 주문 취소

__요구사항 분석__

> - 상품은 제고 관리가 필요하다.
> - 상품의 종류는 도서, 음반, 영화가 있다.
> - 상품을 카테고리로 구분할 수 있다.
> - 상품 주문시 배송 정보를 입력할 수 있다.

__도메인 모델__

![도메인 모델](/assets/img/jpaExercise2/WebApplication개발1%20-%20도메인%20모델.png)

> - 다대일, 일대다, 상속관계를 확인.
> - 상품 분류로는 도서, 음반, 영화



__앤티티 설계__

![엔티티 설계](/assets/img/jpaExercise2/WebApplication개발1%20-%20테이블%20설계.png)

> - 도메인 모델을 참고하여 엔티티 설계.


__테이블 설계__

![테이블 설계](/assets/img/jpaExercise2/WebApplication개발1%20-%20테이블%20설계2.png)

> - 엔티티 설계와 별개의 테이블 설계.

Domain 구현과 연관관계에 대한 코드들은 github에 올려놓았다.

[Entity Domain](https://github.com/sangoun94/jpashop/tree/main/src/main/java/com/jpabook/jpashop/domain)