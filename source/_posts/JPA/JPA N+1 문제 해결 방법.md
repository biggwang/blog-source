---
title: N+1 문제 해결 방법
catalog: true
date: 2019-07-10 23:55:15
subtitle: 
header-img: "bg_computer.jpg"
tags: 
- Learnning
catagories:
- JPA
---

# 예제 Repository 
[류윤광 Github](https://github.com/biggwang/learnning-subjects/tree/master/spring-jpa-n-plus-one)

# 학습목표
- N+1 문제가 무엇인지 안다.
- 그것을 해결하기 위한 방법이 어떤것이 있으면 각 방법에 장단점을 안다.
- 실무에 정확히 적용 할 줄 안다.


# 우선 junit에 테스트 한다면 아래 사항을 주의 하자
- junit은 기본적으로 테스트가 끝나면 rollback 되므로 그 영향으로 JPA에 영속성 컨텍스트에서 실행해야 할 쿼리를 날리지 않을 수 있다 어차피 롤백 될것이기 때문에 @Rollback(false)를 활용 하자
- 하나의 테스트 메소드 내에서 save하고 select하면 pxoy initialize session 에러를 접할 수 있는데 그때는 @Transactional 를 사용하자


# N + 1 문제
- 영속성 컨텍스트에 **지연로딩(Lazy Loading)** 때문에 발생하는 현상이다.
- A와 B테이블이 서로 1 to n 으로 연관관계가 있다고 하자.  
쿼리로 따지면 조인하여 한번에 가져오고 싶은데 실제 사용 할 때마다 연관된 B테이블에 계속 쿼리를 날리는 것이다.
- B테이블에 100개의 row가 있다면 100번을 날리는 것이다.
- 아래를 보자

{% asset_img "jpa_n_plus_one.jpg" %}  

총 11개의 쿼리가 날라갔다.   
- A 테이블 select 1개  
- B 테이블 select 10개

A 테이블만 조회 
~~~ java
List<Teacher> teacherList = teacherRepository.findAll();  
~~~

하지만 B테이블에 정보를 원한건 아니었고 @OneToMany는 FetchType이 기본적으로 Lazy이므로 B테이블 데이터는 쿼리를 날리지 않는다. 대신 프로직시 객체를 가지고 있다.  

이제 실제 B테이블을 사용 하려고 한다.  

~~~ java
 List<String> nameList = teacherList.stream()
    .map(teacher -> teacher.getStudents().get(0).getName())
    .collect(Collectors.toList());
~~~

여기서 teacherList에 갯수가 만큰 select 쿼리가 발생하는 것이다.  
이 부분에서 n+1 문제가 발생했다고 하는 것이다.  
그렇다면 어떤 방법으로 해결 할 수 있을까?




## 해결방법 1 fetch join

~~~ java
public interface TeacherRepository extends JpaRepository<Teacher, Long> {
    @Query("select t from Teacher t join fetch t.students")
    List<Teacher> findAllJoinFetch();
}
~~~

{% asset_img "jpa_fetch_join.jpg" %}  

한번에 가져오기 위해 JPA가 쿼리를 만들어서 가져오게 한다. 이후에 말하겠지만 fetch join 은 inner join을 사용한다.

> TODO QueryDSL로도 fetch join 해보기

### fetch join에 문제점 - 불필요한 쿼리를 발생시킨다.

> TODO fetch join에 한계를 보여 줄 수 있는 코드 작성


## 해결방법 2 @EntityGraph

내가 Eager로 가져오고 싶은것만 가져 올 수 있다.  

~~~ java
public interface TeacherRepository extends JpaRepository<Teacher, Long> {
    @EntityGraph(attributePaths = "students")
    @Query("select t from Teacher t")
    List<Teacher> findAllEntityGraph();
}  
~~~


{% asset_img "jpa_entity_graph.jpg" %}  
 
### @EntityGraph 사용 주의사항 - OuterJoin 그리고 중복
@EntityGraph는 OuterJoin을 사용하기 때문에 중복 데이터가 들어 올수 있는데 그것은 아래와 같이 Set을 통해 중복을 제거 할 수 있다.

~~~ java
@Entity
@Getter
@NoArgsConstructor
@Table(name = "teacher")
public class Teacher {

...

    @OneToMany(fetch = FetchType.EAGER, cascade = CascadeType.ALL)
    @JoinColumn(name="teacher_id")
    private Set<Student> students = new ArrayList<>();   
}
~~~


#### 하지만 어쨌든 불필요한 쿼리
해당 방법은 Java에서 중복을 제거한거고 DB에 불필요한 쿼리를 날린건 똑같다.  
애초에 OuterJoin을 안쓰도록 노력하는게 좋을것 같다.



# 지연로딩과 즉시로딩 개념


## 즉시로딩 (EAGER)
- @ManyToOne 에서 기본값
- 연관된 엔티티에 모든 데이터를 DB 콜하여 한꺼번에 가져온다.

## 지연로딩 (LAZY)
- A 엔티티에 연관된 B 엔티티 정보가 있을때 A 엔티티 정보만 조회 하면 비효율적이므로 로딩을 지연하여 실제 사용할 때 데이터를 조회 한다.
- 실제 사용 할때는 B 엔티티 정보를 사용하고자 할 때 실제 쿼리가 DB에 날리게 된다.
- A 엔티티에서 연관된 B 엔티티 정보는 Proxy 형태로 (디버그로 보면 jvst 문자가 포함됨) 가지고 있다가 실제로 B 엔티티를 사용 할 때 쿼리를 날린다.


## 연관 관계의 엔티티를 어떻게 가져올 것인가 

- @OneToMany 기본값 Lazy
- @ManyToMany 기본값 Eager

Post, Comment 테이블 관계에서 Post에서 Comments데이터를 기본적으로 가져오지 않는다.  
왜냐하면 Comment에 Row가 얼마나 있을지 모르기 때문이다. 그래서 OneToMany 기본값은 Lazy 이다.  

즉, Post에 있는 데이터만 가져오겠다는 뜻이다.

## 프록시

-  엔티티를 조회할 때 영속성 컨텍스트에 존재하지 않으면 DB콜을 하게 되는데 그런 I/O 발생없이 실제 사용하는 시점까지 미루고 싶을때 사용한다.
-  즉, 지연로딩을 사용하기 위해 프록시 객체가 필요함

> Memeber member = entityManager.getReference(Member.class, "member1");


## 참고
- https://yellowh.tistory.com/125
- https://jojoldu.tistory.com/165
