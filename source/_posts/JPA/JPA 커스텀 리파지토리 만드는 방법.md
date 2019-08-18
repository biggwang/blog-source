---
title: JPA 커스텀 리파지토리 만드는 방법
catalog: true
date: 2019-07-25 20:29:24
subtitle: 
header-img: "bg_computer.jpg"
tags: 
- Learnning
catagories:
- JPA
---

# 학습목표
- JPA 커스텀 리파지토리 만드는 방법을 알고 실무에 적절한 상황에 적용하여 생산성과 유지보수 향상을 도모한다.

# 들어가며
해당 설명에 대한 코드는 [Github](https://github.com/biggwang/learnning-subjects/tree/master/spring-jpa-n-plus-one) 에 있습니다.  
내용은 차근 차근 업데이트 하고 살을 붙여 나가도록 하겠습니다.

# 커스텀 리파지토리를 왜 만드는가?

Hibernate, Spring Data 제공하는 기능 외 내가 필요한 기능을 추가하거나 기존에 있는 기능을 오버라이딩하여 데이터를 영속화 시키고 싶을때 사용합니다. 말로는 한계가 있으니 우선 코드부터 보시죠   


# 커스텀 리파지토리 만드는 방법 1

~~~ java
/**
 * Spring JPA에서 제공하는 것 외에 내가 필요한 기능을 직접 정의한다.
 */
public interface StudentCustomRepository<T> {

    // Spring JPA에 없는 기능을 추가
    List<Student> findStudent();

    // Spring JPA에서 제공하는 기능을 재정의
    void delete(T entity);
}
~~~
이렇게 내가 리파지토리 레이어에서 사용하고 싶은 기능들을 인터페이스로 정의합니다.  

 그 다음은 정의한 인터페이스를 implement 합니다.
~~~ java
/**
 * 마치 DAO 랑 비슷한 패턴이다.
 */
@Slf4j
@Repository
@Transactional
public class StudentCustomRepositoryImpl implements StudentCustomRepository {

    @Autowired
    EntityManager entityManager;

    @Override
    public List<Student> findStudent() {
        log.info("custom findStudent");
        return entityManager.createQuery("SELECT s FROM Student AS s", Student.class)
            .getResultList();
    }

    @Override
    public void delete(Object entity) {
        log.info("custom delete");
        entityManager.remove(entity);
    }
}
~~~

Mybatis 사용 경험 있으신분은 DAO 패턴이랑 비슷하게 생각하시면 됩니다.  여기에 객체지향 쿼리를 작성하면됩니다.  위 코드에서는 JPQL을 사용하였지만 Criteria Query DSL을 사용하면 더 객체지향 쿼리를 개발 할 수 있습니다.   

마지막으로 실제 사용할 리파지토리만 만들어 주면 됩니다.  

~~~ java
public interface StudentRepository extends JpaRepository<Student, Long>, StudentCustomRepository<Student> {

}
~~~

확인은 

~~~ java 
@RunWith(SpringRunner.class)
@DataJpaTest
public class StudentRepositoryTest {

    @Autowired
    private StudentRepository studentRepository;

    /**
     * @DataJpaTest 에서는 @Transaction 이 선언되어 있기 떄문에 기본적으로 Rollback이 된다.
     */
    @Test
    public void crud() {
        studentRepository.findStudent();

        Student student = Student.builder().name("홍길동").build();
        studentRepository.save(student);

        // Rollback이 수행 되기 때문에 반영이 안된다. delete를 반영하기 위해 flush를 씀
        studentRepository.delete(student);
        studentRepository.flush();
    }
}
~~~




# 커스텀 리파지토리 만드는 방법 2

이번엔 상황을 예를 들어 보겠습니다. 위 방법데로 커스텀하게 만들어서 잘쓰고 있었는데 A리파지토리 B리파지토리 ... 공통적으로 사용되는 기능들이 있는겁니다.  
위 방법은 하나의 Entity에 한해서 리파지를 커스텀하게 만든것이고 이번에는 모든 리파지토리에 공통기능을 커스텀 리파지토리를 만들고 싶은 것입니다.  

코드를 보죠  

~~~ java
/**
 * Repository 공통 기능 정의
 *
 * @param <T> Entity
 * @param <ID> Key
 */
@NoRepositoryBean   // 중간자 repository
public interface MyRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {

    boolean contains(T entity);
}
~~~

모든 리파지토리에 공통 기능을 적용시키고 싶을 리파지토리를 정의합니다.  

**TODO: @NoRepositoryBean에 대해서 설명하기**


이제 위 정의한 구현체를 만듭니다.  
보시면 SimpleJpaRepository를 상속 받았는데요 SimpleJpaRepository 는 JPA에서 클래스 상속구조에서 제일 밑단에 있는 구현 리파지토리 구현체입니다. 모든 공통기능을 가진 리파지토리를 만들고 있는 중이므로 기능이 많은 놈을 상속 받는게 낫겠죠??  

그리고 아까 내가 공통으로 넣고 싶은 커스텀 리파지토리 또한 같이 넣어 줍니다.

~~~ java
public class SimpleMyRepository<T, ID extends Serializable> extends SimpleJpaRepository<T, ID> implements MyRepository<T, ID> {

    private EntityManager entityManager;

    public SimpleMyRepository(JpaEntityInformation<T, ?> entityInformation,
        EntityManager entityManager) {
        super(entityInformation, entityManager);
        this.entityManager = entityManager;
    }

    @Override
    public boolean contains(Object entity) {
        return entityManager.contains(entity);
    }

}
~~~


그 다음 Spring Boot에게 커스텀하게 만든 리파지토리가 있다는 것을 알려주기만 하면 됩니다.
~~~ java
/**
 * @EnableJpaRepositories(repositoryImplementationPostfix = "Default") 커스터 리파지토리 Postfix "impl" 이 맘에
 * 안 들때
 */
@SpringBootApplication
@EnableJpaRepositories(repositoryBaseClass = SimpleJpaRepository.class)
public class SpringJpaDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringJpaDemoApplication.class, args);
    }

}
~~~


실제 리파지토리에서 사용만 하면 끝~!

~~~ java
public interface SubjectRepository extends MyRepository<Subject, Long> {

}
~~~

