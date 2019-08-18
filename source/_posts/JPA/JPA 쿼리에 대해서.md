---
title: JPA 쿼리에 대해서
catalog: true
date: 2019-07-23 10:05:15
subtitle: 
header-img: "bg_computer.jpg"
tags: 
- Learnning
catagories:
- JPA
---

# 학습목표
- JPA 쿼리를 왜 사용하는지 이해한다.
- JPA 쿼리 종류가 어떤것이 있으며 각 장/단점을 각 상황에 따라 실무에 적용 할 줄 안다.


# JPA 쿼리!?!? 쿼리를 안써도 되는 줄 알았는데??

JPA를 공부하면서 가장 큰 착각 중에 하나였다.  
아직도 의문점이 드는게 있고 완벽하진 않지만 하나하나 궁금증을 해결해 가면서 지식을 내것으로 만들어 보겠다.  

## 패러다임 불일치 해결

JPA를 입문할 때 가장 많이 들었던 용어중 하나 였다.  
의미는 객체지향과 관계지향에 성격이 다른 것을 ORM을 통해 그 불일치를 해결해 준다는 것이다. 나같은 경우는 JAVA 와 RDBMS 였다.  

## 그럼 왜 패러다임 불일치를 해결 할까?

사실 내가 여태껏 웹 개발을 경험하면서 대부분에 비지니스 로직에 초점은 결국 쿼리를 어떻게 작성하느냐고 그 쿼리에 맞춰 DAO, Service, Controller 순으로 개발 하였다.  
JPA 를 쓰기전에는 Mybatis를 사용하였고 각 기능마다 CRUD 쿼리를 작성해주어야 했다.  
내가 말하고 싶은 포인트는 자바 OOP로 개발해야 소프트웨어가 변경에 flexible하고 쉬운 유지보수를 할 수 있는데 쿼리 중심으로 맞춰 개발 하다 보니 객체지향 프로그래밍에 장점을 살리지 못하고 있는 것이다.

어쨋든 프로그래밍을 하면서 데이터는 RDB에 넣긴 해야 한다.  넣어야 하는데 그것을 테이블간에 관계도를 생각하면서 쿼리를 작성하는데 초점을 두는것이 아니라 객체간에 관계 참조를 쉽게 코드로 표현 하고 그 표현을 해석하는 ORM, Hibernate, JPA, 영속성 컨텍스트가 감지하여 쿼리를 만들어 DB에 영속화 시키는 것이다.  

이런 중간 역활을 ORM을 함으로써 개발 할 때, 데이터를 영속화 할 때 객체지향으로 개발할 수 있다는점에서 패러다임 불일치를 해결 한다고 말한다고 생각한다.


다시 돌아서 제일 처음 궁금증에 대해서 다시 생각해 보면,  ORM, JPA가 패러다임 불일치를 해결한다고 했으니까 나는 쿼리를 작성안하고 객체지향 프로그래밍을 하면 영속성 컨텍스트에서 알아서 쿼리를 만들고 DB에 쿼리를 날리는 것을 생각 하였다.  

하지만 일부만 맞다고 생각이 든다. 왜냐하면 단순히 저장(persist), 수정(객체 속성값 변경), 조회(findAll), 삭제(delete) 기능이라면 쿼리를 작성 할 필요도 없다.  
JPA가 제공해주는 정확히 Hibernate 가 제공해주는 API를 사용함으로써 **단순 CRUD에 대한 쿼리 작성은 필요 없는것이다.**  

단순 CRUD라고 하였고 **그럼 복잡한 CRUD는 ??** 그때 바로 필요 한게 JPA에서 사용하고 있는 여러 쿼리 작성이 필요 하다고 생각하고 그래서 존재한다고 생각한다.  
복잡한 애플리케이션 일수록 복잡한 쿼리를 요구 하게 되므로 위 간단한 API 호출로는 한계가 있다. 

쿼리라 말하였지만 정확히 객체지향 쿼리이다.  
하지만 이부분은 아직 잘 공감되지 않는 부분이기도 한데, 객체지향 쿼리라고 하지만 객체지향 쿼리도 모양은 결국 쿼리다.  하지만 객체지향 쿼리라고 말하는 이유는 쿼리를 만드는데 타켓이 데이터베이스에 있는것이 아닌 바로 객체 Entity를 가지고 쿼리를 만들기 때문이다. 

# JPA 쿼리 종류 및 장.단점

가장 중요한것은 JPQL이다.  Criteria 이나 Query DSL은 JPQL을 편리하게 사용 하도록 도와주는 기술일 뿐이다.  
객체지향 쿼리는 DBMS 종류에 관계없이 추상화된 쿼리이니 DBMS를 신경쓰지 않아도 되는 큰 이점이 있다.  본격적으로 쿼리 종류를 알아보면

## JPQL(HQL)

~~~ java
String jpql = "select m from Member as m where m.username = 'ryu'";
List<Member> resultList = 
    em.createQuery(jpql, Member.class).getResultList();
~~~

하지만 문자열로 저렇게 작성하면 오타가 나도 컴파일시점이 아닌 런타임 시점에 알 수 있어 외부 변경에 취약하다.

## Criteria

JPQL을 생성하는 빌더 클래스로 Type Safe하게 사용 할 수 있다. (JPA 2.0 부터 지원)

~~~ java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

Root<Member> m = query.from(Member.class);

CriteriaQuery<Member> cq = 
    query.select(m).where(cb.equal(m.get("username"), "kim"));
List<Member> resultList = em.createQuery(cq).getResultList();
~~~

Type Safe 하게 쿼리를 작성하는 것 말고는 코드가 복잡해 지는 단점이 있다.

## Query DSL 

Criteria 와 마찬가지로 JPQL 빌더 역활을 하지만 Criteria 보다 간단하고 사용하기 쉽다.

~~~ java
JPAQuery query = new JPAQuery(em);
QMember member = QMember.member;

List<Member> members = 
    query.from(member)
    .where(member.username.eq("kim"))
    .list(member);

~~~

## Native Query

특정 DB에서 제공하는 기능을 꼭 쓰고 싶을때 사용한다. 예를 들어 ORACLE DB만 제공하는 CONNECT BY 등이다.  

~~~ java
String sql = "SELECT ID, NAME FROM MEMBER WHERE NAME = 'kim'";
List<Member> resultList = 
    em.createNativeQuery(sql, Member.class).getResultList();
~~~