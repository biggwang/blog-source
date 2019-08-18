---
title: JPA 영속성 컨텍스트 엔티티 관리 방법
catalog: true
date: 2019-07-27 17:35:55
subtitle: 
header-img: "bg_computer.jpg"
tags: 
- Learnning
catagories:
- JPA
---

# 학습목표
- 영속성 컨텍스트가 엔티티를 생성,수정,삭제 하는 동작을 파악하여 실무 개발할때 삽질을 줄이고 불필요한 쿼리를 발생시키지 않게 한다.
- JPA를 활용하여 성능향상을 시킨다.

# 들어가며
관련 샘플 코드는 [Github](https://github.com/biggwang/learnning-subjects/tree/master/jpa-persistcontext)에 있습니다.  
영속성 컨텍스트에 대한 엔티티 관리를 다양한 예제를 만들어보고 피드백을 받아 앞으로 계속 업데이트를 하도록 하겠습니다.
 

# 1차 캐시
~~~ java
Member member = new Member();
member.setId("member1");
member.setUserName("회원1");

// 1차 캐쉬 저장, 영속성 컨텍스트에 저장 됨
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
em.persist(member);

// 1차 캐쉬 조회
Member findMember = em.find(Member.class), "member1");

em.getTransaction().end();
~~~
 
DB로 가기전에 먼저 영속성 컨텍스트에 "member1"에 해당하는 데이터가 있으면 우선 조회 한다.  
단, 한 트랜잭션내에서만 1차 캐쉬가 보관되어 지는 것이다. 또 동시에 100명에 접속자가 있다면 100개에 영속성 컨텍스트가 생기는 것이며 서로 공유 되지 않는다. 
 
그러면 1차 캐쉬에 없는 데이터는 어떻게 될까??  
그때는 DB에서 조회하고 가져온 데이터를 1차 캐쉬에 저장하고 그리고 그 값을 반환한다.

# 동일성 보장

~~~ java
Member a = em.find(Member.class), "member1");
Member b = em.find(Member.class), "member1");

a == b 하면 true 즉, heap 같은 주소를 가지고 있는 객체라는 뜻인데 1차 캐쉬에 보관 된 데이터를 반환하였기 때문에 같은 주소를 가진 데이터 인것이다.

~~~

# 쓰기 지연

DB 트랜잭션 개념이다. 
엔티티매니저에 의해서 관리되고 있는 엔티에서 쿼리를 막날려도 DB에 날라가지 않는다. 왜냐면 트랜잭션 상태에서 아직 커밋되지 않았기 때문이다.  
flush를 하거나 트랜잭션이 끝나면 그때 DB에 반영된다. 정확히 말하면 트랜잭션이 끝나기전에 JPA가 flush를 하여 영속성 컨텍스트 관리되는 엔티티와 DB와 비교후 변경된것이 있다면 쿼리를 생성하여 쓰기지연 SQL 저장소에 보관하고 그 쿼리를 DB에 반영하고 트랜잭션이 끝나면 커밋이 되어 최종 반영 되는 것이다.

~~~ java
@Test
public void 쓰기지연_INSERT_동작_테스트() {
    EntityManager em = emf.createEntityManager();
    EntityTransaction transaction = em.getTransaction();

    transaction.begin();

    Teacher teacher1 = Teacher.builder().classGroupName("2학년 1반").build();
    em.persist(teacher1);
    System.out.println("teacher1 저장?");
    
    Teacher teacher2 = Teacher.builder().classGroupName("2학년 2반").build();
    em.persist(teacher2);
    System.out.println("teacher2 저장?");
    
    Teacher teacher3 = Teacher.builder().classGroupName("2학년 3반").build();
    em.persist(teacher3);
    System.out.println("teacher3 저장?");

    System.out.println("여기서 실제 쿼리 날리지롱~! commit 전");
    transaction.commit();
    System.out.println("여기서 실제 쿼리 날리지롱~! commit 후");
}
~~~

{% asset_img "write-lazy.png" %}


## 객체지향 쿼리(JPQL, Criteria, QueryDSL) 사용시에는 flush가 발생
밑에 코드에서 query.getResultList() 를 하는 순간 영속화(perist) 3번 하였던 것이 우선 insert를 3번 날리어 DB에 반영하게 됩니다.  
왜냐면 그다음 코드에서 select 구문을 날리는데 아무리 쓰기지연이라 하지만 이전에 3개 insert 하는것이 반영이 안되어 select 하는게 의미가 있을까요?? 그래서 JPA에서는 객체지향 쿼리를 사용하게 되면 바로 DB에 반영이 됩니다.

~~~ java
@Test
public void JPQL_사용시_자동_Flush_수행_동작_테스트() {
    EntityManager em = emf.createEntityManager();
    EntityTransaction transaction = em.getTransaction();

    transaction.begin();

    Teacher teacher1 = Teacher.builder().name("홍길동").age(32).classGroupName("1학년 1반").build();
    em.persist(teacher1);
    System.out.println("홍길동 선생님이 등록되었습니다.");

    Teacher teacher2 = Teacher.builder().name("임꺽정").age(15).classGroupName("2학년 7반").build();
    em.persist(teacher2);
    System.out.println("임꺽정 선생님이 등록되었습니다.");

    Teacher teacher3 = Teacher.builder().name("성춘향").age(27).classGroupName("3학년 5반").build();
    em.persist(teacher3);
    System.out.println("성춘향 선생님이 등록되었습니다.");

    Query query = em.createQuery("select t from Teacher t", Teacher.class);

    System.out.println("객체지향 쿼리를 사용하여 flush를 수행합니다. 1");
    List<Teacher> teacherList = query.getResultList();

    transaction.commit();
}
~~~
{% asset_img "jpql-flush.png" %}


# 변경 감지(Dirty Checking)
**영속성 컨텍스트에서 관리되는 엔티티인 경우에 엔티티 정보가 변경 되었을때 변경 감지**가 되어 업데이트 쿼리가 만들어집니다. 만들어 지는 과정은 이렇습니다.

- 1차 캐쉬에 존재하는 엔티티와 변경 여부 확인
- 값이 다르다면 UPDATE 구문을 만들고 쓰기 지연 SQL 저장소에 보관
- flush 하여 DB 쿼리 날림
- commit 되면 최종 반영 


## UPDATE 구문은 기본적으로 모든 컬럼을 UPDATE 한다.

예를 들어 컬럼이 30개 라면 컬럼 30개에 대한 모든 컬럼을 업데이트 한다.  

~~~ java
@Test
public void 변경감지_동작_테스트() {
    EntityManager em = emf.createEntityManager();
    EntityTransaction transaction = em.getTransaction();

    transaction.begin();

    em.persist(Teacher.builder().classGroupName("2학년 1반").build());
    Teacher teacher1 = em.find(Teacher.class, 1L);

    System.out.println("원래 맡은 반: " + teacher1.getClassGroupName());

    teacher1.setClassGroupName("3학년 1반");
    System.out.println("새롭게 맡은 반: " + teacher1.getClassGroupName());

    transaction.commit();
}
~~~

{% asset_img "dynamic-update2.png" %}

모든 컬럼을 업데이트 치는 것이 불필요하다 생각 할 수 있으나 아래와 같은 이점이 있다고 한다.
- 모든 컬럼을 업데이트 하기 때문에 쿼리가 항상 같아서 DB에 동일한 쿼리를 보낼테고 이는 DB가 쿼리를 캐슁하여 캐슁한 쿼리를 계속 사용 할 수 있다.  

컬럼이 30개이상 되는 엔티티라면 기본값인 모든 컬럼을 업데이트 치는 것보다 @DynamincUpdate 을 사용하여 변경된 컬럼만 업데이트 치는것이 낫다고 한다.


## 일부만 UPDATE 하고 싶다면?
위에서 언급하였지만 @DynamincUpdate 이용해 @Entity 선언한 부분에 넣어주면 되고 위 코드에서 teacher1.setClassGroupName("3학년 1반"); 하게 되면 "classGroupName" 컬럼만 업데이트 하게 된다.

{% asset_img "dynamic-update1.png" %}



# [문제] DB에 날리는 쿼리는 몇 번 호출 되었을까?
~~~ java

Account account = new Account();
account.setUsername("ryu");
account.setPassword("1111");

Study study = new Study();
study.setName("Spring Data JPA");

account.addStudy(study);

Session session = entityManager.unwrap(Session.class);
session.save(account);
session.save(study);

Account yungwang = session.load(Account.class, account.getId());
yungwang.setUsername("gwang");
yungwang.setUsername("boriswinter");
yungwang.setUsername("ryu");

~~~


답은 insert 2번이다.  

session.save(account);
session.save(study);  

이부분이 2번 insert 쿼리가 DB로 날라간것이다. 그럼 setUserName 메소드 3번 호출한건 왜 Update가 되지 않지??

바로 이 부분이 Psersist Context에 성능 이점이다.  
코드를 보면 ryu -> gwang -> boriswinter -> ryu 결국 name은 변하지 않았기 때문에 JPA가 알아서 쿼리를 날리지 않았던 것이다.  
이부분이 Dirty Checking 했다는 것이고 DB에 쿼리 날리는 것을 최소화 한다는 개념 Write Behind 개념이다.


# TODO
- 


