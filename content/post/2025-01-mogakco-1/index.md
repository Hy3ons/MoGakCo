---
title: 하계 모각코 01
description: 2025년-하계 첫번째 모각코 게시글
slug: 2025-01-mogakco-1
date: 2025-07-14 20:45:00+0900
image: cover.jpg
author: "황현석"
categories:
    - spring
weight: 1
---

# 하계 모각코 01 - 목표

이번 모각코 시간에는 유데미 강의를 들으며, Spring Boot와 JPA, Hibernate, Spring JDBC 등 다양한 주제를 공부할 예정입니다.  
특히, 프로덕션 환경 배포 준비, Actuator, Spring MVC, JPA와 Hibernate, H2 콘솔, Spring JDBC 등 여러 내용을 주제로 하는 영상을 시청 할 예정입니다.

저는 모각코 시간에만 공부하는 것이 아니다 보니, 그때그때 관심사나 공부의 진도, 필요에 따라 주제가 달라질 수 있습니다.  
이번 시간에는 아래와 같은 강의들을 목표로 삼고, 각 단계별로 짧게라도 정리해보려 합니다.

## 오늘의 목표 강의 목록

1. Spring Boot로 프로덕션 환경 배포 준비하기 (Embedded Server, Actuator)
2. Spring Boot, Spring, Spring MVC 이해하기
3. Spring Boot 시작하기 - 복습
4. JPA와 Hibernate 시작하기 및 새 프로젝트 설정
5. H2 콘솔 실행 및 과정 테이블 생성
6. Spring JDBC 시작 및 데이터 삽입/삭제/쿼리 실습
7. JPA 실습

짧은 시간 동안 여러 주제를 다루겠지만, 각 단계별로 핵심만 정리해보는 것이 오늘의 목표입니다.

# 하계 모각코 01 - 결과

## 1. Maven을 이용한 빌드와 실행

- **Maven 명령어 활용법**
  - `mvn clean` : 기존 빌드 산출물(클래스, jar 등)을 삭제
  - `mvn install` : 컴파일 → 테스트 → 패키징 → 로컬 Maven 저장소에 배포
- **Spring Boot 실행**
  - 빌드된 `.jar` 파일을 커맨드라인에서 실행 가능  
  - ```bash
    java -jar target/myapp.jar
    ```


## 2. Spring Boot Actuator를 통한 모니터링

- Spring Boot Actuator는 애플리케이션 상태, 메트릭, 헬스 체크 등 다양한 모니터링 기능을 제공합니다.
- 주요 엔드포인트 예시  
  - `/actuator/health` : 서비스 상태 확인
  - `/actuator/metrics` : 메트릭 정보 제공
- 보안 설정 필수 (민감 정보 노출 방지)
- actuator 엔드포인트 노출 설정 (기본은 제한적임)

```properties
management.endpoints.web.exposure.include=health,metrics,info
```
    

## 3. DB 연동 및 SQL 활용

- **메모리 기반 DB 사용**  
  - H2 같은 인메모리 DB로 빠른 개발과 테스트 가능, 강의영상에는 H2 SQL을 사용해서, 실습하였다.
- **JDBC vs Spring JDBC vs JPA 비교**  
  | 구분       | 특징                                      |
  |------------|-------------------------------------------|
  | JDBC       | 순수 자바 DB 연동, SQL 직접 작성, 번거로움    |
  | Spring JDBC| JDBC API 래핑, 예외 변환 및 편의 메서드 제공, SQL 직접 작성해야한다ㅓ  |
  | JPA        | 객체와 테이블 매핑, ORM 제공, 복잡한 DB 작업 간결화 |


## 4. Spring Data JPA 기본 사용법

- `JpaRepository` 인터페이스 상속만으로 CRUD 자동 구현해준다. 
- Spring Boot가 런타임 시점에 **인터페이스 구현체(프록시)를 생성하여 Bean으로 등록**  
- 메서드 이름만으로 쿼리 자동 생성 가능  
  예) `findByName`, `findByCreatedAtAfter` 등

- 직접 쿼리를 작성하고 싶으면 `@Query` 어노테이션을 사용할 수 있다.
-   ```java
    public interface CourseRepository extends JpaRepository<Course, Long> {

        // JPQL 직접 작성
        @Query("SELECT c FROM Course c WHERE c.name = :name")
        List<Course> findByNameCustom(@Param("name") String name);

        // 네이티브 쿼리 사용 (nativeQuery=true)
        @Query(value = "SELECT * FROM course WHERE created_at > :date", nativeQuery = true)
        List<Course> findRecentCoursesNative(@Param("date") LocalDate date);
    }

## 5. 복잡한 로직 확장 방법

### 5.1 Custom Repository 구현

- 복잡한 쿼리나 동적 쿼리 작성 시 별도의 커스텀 인터페이스와 구현체 작성 필요
- 예시  
  ```java
  public interface CourseCustomRepository {
      List<Course> findRecentCourses();
  }

  @Repository
  public class CourseCustomRepositoryImpl implements CourseCustomRepository {
      @PersistenceContext
      private EntityManager em;

      public List<Course> findRecentCourses() {
          // JPQL 직접 작성
          return em.createQuery("SELECT c FROM Course c WHERE c.createdAt > :date", Course.class)
                   .setParameter("date", LocalDate.now().minusDays(7))
                   .getResultList();
      }
  }

  public interface CourseRepository extends JpaRepository<Course, Long>, CourseCustomRepository {
  }

## 6. EntityManager와 `@PersistenceContext`

- EntityManager는 JPA의 핵심으로,  
  영속성 컨텍스트(1차 캐시)와 트랜잭션 상태를 관리하는  
  **상태가 있는 객체**입니다.

- `@PersistenceContext`를 사용해 주입하며,  
  내부적으로는 트랜잭션마다 별도의 EntityManager 인스턴스를  
  **프록시를 통해 트랜잭션 컨텍스트에 맞게 연결해줍니다.**

- 직접 싱글톤 Bean으로 쓰면 상태 공유로 인한  
  **동시성 문제와 상태 꼬임이 발생할 수 있으니 주의해야 합니다.**

- 따라서, @AutoWired를 사용하면 안 됩니다.


## 7. Hibernate와 JPA

- **JPA**는 Java ORM의 표준 인터페이스(스펙)입니다.

- **Hibernate**는 JPA 스펙을 구현한  
  가장 널리 쓰이는 ORM 구현체(라이브러리)입니다.

- Spring Boot는 기본적으로 Hibernate를  
  JPA 구현체로 사용하여 ORM 기능을 제공합니다.



# 강의 외 추가적인 공부 : 다중 데이터소스 관리

하나의 Spring Boot 애플리케이션에서 여러 개의 데이터베이스를 사용하고,  
각 데이터베이스에 연결된 JPA Repository를 분리하고 싶을 때,  
멀티 데이터소스 설정이 필요합니다.


## 1. 멀티 데이터소스 설정 개요

- 각 데이터베이스별로 `DataSource` 빈을 생성  
- 각각에 맞는 `EntityManagerFactory` 빈 생성  
- `TransactionManager`도 데이터소스별로 분리  
- `@EnableJpaRepositories`를 이용해 Repository 패키지별로 연결 분리

## 2. 각각의 DB 설정 (PrimaryDataSourceConfig.java)

```java
@Configuration // 이 클래스는 Spring 설정 클래스임을 나타냄
@EnableTransactionManagement // @Transactional 애노테이션 기반의 트랜잭션 기능을 활성화함
@EnableJpaRepositories(
    basePackages = "com.example.primaryRepo",    // 이 경로 하위에 있는 Repository 인터페이스들이 이 데이터소스를 사용하도록 설정
    entityManagerFactoryRef = "primaryEntityManagerFactory", // 사용할 EntityManagerFactory Bean 이름
    transactionManagerRef = "primaryTransactionManager" // 사용할 트랜잭션 매니저 Bean 이름
)
public class PrimaryDataSourceConfig {

    /**
     * primaryDataSource: 첫 번째 데이터베이스와 연결할 DataSource를 생성.
     * application.properties 파일에서 'spring.datasource.primary' 접두어로 시작하는 설정을 읽어들임.
     * 예: spring.datasource.primary.url, username, password, driver-class-name 등
     */
    @Bean
    @Primary // 여러 DataSource 중 기본(우선순위)으로 사용됨
    @ConfigurationProperties(prefix = "spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    /**
     * primaryEntityManagerFactory: JPA에서 사용할 EntityManagerFactory를 생성.
     * - 이 팩토리는 해당 DataSource로 DB 연결을 수행하며,
     * - 지정한 Entity 클래스들이 어떤 DB와 연동될지 결정한다.
     * - 'packages()'에 명시된 경로 하위의 Entity 클래스들을 스캔한다.
     * - persistenceUnit 이름은 구분용이므로 임의로 설정 가능함.
     */
    @Bean
    @Primary
    public LocalContainerEntityManagerFactoryBean primaryEntityManagerFactory(
            EntityManagerFactoryBuilder builder) {
        return builder
                .dataSource(primaryDataSource()) // 위에서 만든 DataSource 사용
                .packages("com.example.primaryentity") // JPA Entity 클래스가 위치한 패키지
                .persistenceUnit("primary") // persistence-unit 이름 (JPA 설정의 논리적 이름)
                .build();
    }

    /**
     * primaryTransactionManager: 해당 EntityManagerFactory를 사용하는 트랜잭션 관리자 생성.
     * - @Transactional 동작 시, 이 트랜잭션 매니저가 사용됨.
     * - 하나의 데이터소스에 하나의 트랜잭션 매니저가 대응되어야 함.
     */
    @Bean
    @Primary
    public PlatformTransactionManager primaryTransactionManager(
            @Qualifier("primaryEntityManagerFactory") EntityManagerFactory emf) {
        return new JpaTransactionManager(emf); // 해당 EntityManagerFactory에 대한 트랜잭션 관리
    }
}
```

궁금해서 따로 알아본 내용이며, 실제로 데이터 소스를 여러군대에서 가져와, 활용하는 방식은 보안상에 좋을 순 있지만, 복잡성을 유발하는 것 같아서, 알아만 두자.
