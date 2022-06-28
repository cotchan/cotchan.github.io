---
layout: post
title: 멀티 모듈과 @EntityScan, @EnableJpaRepositories
subtitle: 멀티 모듈에서 싱글 DB, 멀티 DB 사용에 따라 달라질 수 있는 설정
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.

### 포스팅 작성 배경

- 멀티 모듈로 작업을 하게 되면 library 모듈과 서비스 모듈이 분리되므로 Entity, Repository 사용에 있어 바뀌는 부분이 생깁니다.
- 싱글 DB일 때와 멀티 DB인 경우로 나뉠 수 있는데 이 부분을 정리하고자 작성합니다.

### 하나의 DB만 사용하는 경우

- 멀티 모듈에서 하나의 DB만 접근해서 사용하는 경우입니다.
- 하나의 DB만 사용하는 경우 별도의 트랜잭션 매니저를 정의하지 않아도 됩니다.
- 즉, 별도로 `PlatformTransactionManager` 빈을 정의하지 않아도 됩니다.
- 그러나 멀티 모듈이므로 `@EntityScan`, `@EnableJpaRepositories` 범위를 재정의해줘야 합니다.

#### @EntityScan

- application이, 전혀 다른 패키지에 있는 Entity Class를 찾을 수 있도록 해주는 어노테이션입니다.
- 멀티 모듈이므로 엔티티가 library 모듈에 있기 때문에 `basePackages` 설정을 해줘야 합니다.

#### @EnableJpaRepositories

- JPA Repository 빈을 활성화하는 어노테이션입니다.
- 멀티 모듈이므로 레포지토리도 library 모듈에 있기 때문에 `basePackages` 설정을 해줘야 합니다.

#### 코드 예시

- 서비스 모듈의 main 클래스입니다.

```java
// modulecommon: 라이브러리 모듈
// moduleapi: 서비스 모듈
@EnableJpaAuditing
@EntityScan(basePackages = {"com.ac.modulecommon"})
@EnableJpaRepositories(basePackages = {"com.ac.modulecommon"})
@SpringBootApplication(scanBasePackages = {"com.ac.modulecommon", "com.ac.moduleapi"})
public class ModuleApiApplication {

    public static void main(String[] args) {
        SpringApplication.run(ModuleApiApplication.class, args);
    }

}
```

### 2개 이상의 DB를 사용하는 경우

- 멀티 모듈에서 여러개 DB를 접근해서 사용하는 경우입니다.
- 이 경우는 트랜잭션매니저를 여러개 사용해야 하기 때문에 보통 `DataSource`, `PlatformTransactionManager` 빈을 직접 정의해서 사용합니다.
- 그러므로 `PlatformTransactionManager` 빈 설정에 접근할 Entity, Repository 설정이 함께 들어갑니다.

#### 코드 예시

```java
@Configuration
@EnableJpaRepositories(
    basePackages = "YOUR_REPOSITORY_PACKAGE_NAME",
    entityManagerFactoryRef = "apiEntityManager",
    transactionManagerRef = "apiTransactionManager"
)
public class ApiDbConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.hikari")
    public DataSource apiDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties(prefix = "spring.jpa")
    public JpaProperties jpaProperties() {
        return new JpaProperties();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean apiEntityManager(EntityManagerFactoryBuilder builder,
                                                                   DataSource dataSource,
                                                                   JpaProperties jpaProperties) {
        return builder
                .dataSource(dataSource)
                .properties(jpaProperties.getProperties())
                .packages("YOUR_ENTITY_PACKAGE_NAME")
                .persistenceUnit("default")
                .build();
    }

    @Primary
    @Bean(name = "apiTransactionManager")
    public PlatformTransactionManager apiTransactionManager(LocalContainerEntityManagerFactoryBean entityManagerFactory) {
        JpaTransactionManager transactionManager = new JpaTransactionManager(entityManagerFactory.getObject());
        return transactionManager;
    }
}

- application.yml

```yml
spring:
  config:
    activate:
      on-profile: local
  datasource:
    hikari:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:tcp://localhost/~/daangn;MODE=MYSQL;DB_CLOSE_DELAY=-1
      username: sa
      password:
  jpa:
    properties:
      hibernate:
        show_sql: true
        format_sql: true
        use_sql_comments: true
        implicit_naming_strategy: org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
        physical_naming_strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
```
