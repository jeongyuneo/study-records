# Join vs Fetch Join

## 0. 들어가며

JPA를 사용하다보면 N+1 문제를 마주치게 되는데, 이를 해결할 수 있는 방법 중 하나가 Fetch Join이다.

그렇다면 일반적으로 사용하는 Join과 Fetch Join은 어떤 점이 다른지 알아보기 위해 해당 글을 작성하게 되었다.

## 1. Join vs Fetch Join

일반 Join은 **조회의 주체가 되는 엔티티만 SELECT해서 영속화**하고, 연관된 엔티티는 영속화하지 않는다.

반면 Fetch Join은 **조회의 주체뿐만 아니라 연관된 엔티티까지 모두 SELECT하여 영속화**한다.

Fetch Join을 사용하면 연관된 엔티티가 모두 영속화되기 때문에 FetchType이 Lazy인 엔티티를 참조해도 이미 영속성 컨텍스트에서 관리하고 있어 SELECT문이 실행되지 않아 **N+1문제를 해결**할 수 있다.

## 2. Verification

간단하게 Querydsl을 이용해 소속된 team 을 포함한 특정 member의 정보를 반환하는 API를 작성해보자.

![](https://velog.velcdn.com/images/minide/post/715dc136-c880-4e68-a4a1-7124c8eb72da/image.png)

우선, Team A 안에 Member 1과 Member 2가 소속되어 있다고 할 때, Member 1에 대해 조회한다고 하자.

### 2.1. Teck Stack

> Java 11
>
> Spring Boot 2.7
>
> Gradle 8.3
>
> Querydsl 5.0

### 2.2. Setting Querydsl

2.2.1. build.gradle 수정

| build.gradle

```groovy
buildscript {
    ext {
        queryDslVersion = "5.0.0"
    }
}

plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.17'
    id 'io.spring.dependency-management' version '1.1.3'
    id 'com.ewerk.gradle.plugins.querydsl' version '1.0.10'// Querydsl 플러그인 추가
}

...

dependencies {
    ...

// Querydsl 의존성 추가
    implementation 'com.querydsl:querydsl-jpa:5.0.0'
    implementation 'com.querydsl:querydsl-apt:5.0.0'
}

...

def querydslDir = "$buildDir/generated/querydsl"

querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}

sourceSets {
    main.java.srcDir querydslDir
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
    querydsl.extendsFrom compileClasspath
}

compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
```

2.2.2. Q파일 생성

gradle 변경사항 반영 후 IDE 좌측 Gradle > {프로젝트명} > Tasks > other > compileQuerydsl 실행

![](https://velog.velcdn.com/images/minide/post/2305fb2d-3347-4ab1-a467-992d836ef32a/image.png)

querydslDir로 지정했던 위치에 Q파일 생성되면 Querydsl을 사용할 준비가 완료되었다.

2.2.3. QuerydslConfig 생성

JPAQueryFactory를 사용하기 위해서는 EntityManager를 주입해주어야 한다. Querydsl을 사용하는 클래스에서 매번 EntityManager를 주입해주는 일은 굉장히 번거롭기 때문에, 아래와 같이 이를 주입받은 JPAQueryFactory를 Bean으로 등록해서 사용할 수 있다.

| QuerydslConfig.java

```java
@Configuration
public class QuerydslConfig {

    @Bean
    public JPAQueryFactory jpaQueryFactory(EntityManager entityManager) {
        return new JPAQueryFactory(entityManager);
    }
}
```

### 2.3. Implementation

| MemberService.java

```java
@RequiredArgsConstructor
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    @Transactional
    public MemberTeamDto searchMemberWithTeam(Long id) {
        Member member = memberRepository.findMemberWithTeam(id);
        return new MemberTeamDto(
                member.getId(),
                member.getUsername(),
                member.getAge(),
                member.getTeam().getId(),
                member.getTeam().getName()
        );
    }
}
```

MemberService에서는 MemberRepository에서 받아온 Member의 필드와 연관관계 엔티티인 Team의 필드를 통해 MemberTeamDto를 생성해 반환한다.

2.3.1. Finding Member With Join

| MemberRepository.java

```java
@RequiredArgsConstructor
public class MemberRepositoryImpl implements CustomMemberRepository {

    private final JPAQueryFactory queryFactory;

    public Member findMemberWithTeam(Long memberId) {
        return queryFactory
                .selectFrom(member)
                .join(member.team, team)
                .where(member.id.eq(memberId))
                .fetchOne();
    }
}
```

일반 join을 이용하면 호출하는 엔티티만 영속화하기 때문에, FetchType이 Lazy인 엔티티는 이를 참조할 때 SELECT문이 실행되게 된다.

![](https://velog.velcdn.com/images/minide/post/9d49c927-fa48-41b1-963d-0655f7e9afa9/image.png)

2.3.2. Finding Member With Fetch Join

| MemberRepository.java

```java
@RequiredArgsConstructor
public class MemberRepositoryImpl implements CustomMemberRepository {

    private final JPAQueryFactory queryFactory;

    public Member findMemberWithTeam(Long memberId) {
        return queryFactory
                .selectFrom(member)
                .join(member.team, team)
                .fetchJoin()// 추가된 부분
                .where(member.id.eq(memberId))
                .fetchOne();
    }
}
```

fetchJoin을 사용하면 FetchType이 Lazy인 엔티티를 참조해도 이미 영속성 컨텍스트에 저장된 상태이기 때문에 아래와 같이 SELECT문이 나가지 않는 것을 확인할 수 있다.

![](https://velog.velcdn.com/images/minide/post/9afb7ed6-a107-434c-b84f-00b6854b3d63/image.png)

## 3. When to Use Each Join

그렇다면, Fetch Join을 이용하는게 항상 성능상 이점을 줄 수 있을까?

항상 그렇지는 않다.

예를 들어 쿼리 검색 조건에는 필요하지만 실제 데이터는 필요하지 않은 경우, Join을 이용해 검색 조건을 이용하되 해당 조건 검색에 사용된 연관 엔티티를 영속화하지 않을 수 있다.

이렇듯 각 Join문의 특징을 이해하고 상황에 맞춰 적절히 사용함으로써 JPA의 성능을 최적화할 수 있다.
