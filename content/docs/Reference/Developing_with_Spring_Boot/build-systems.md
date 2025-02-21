---
title: Build Systems
date: 2024-02-21
weight: 2
---

## 빌드 시스템

의존성 관리를 지원하고 Maven Central 저장소에 게시된 아티팩트를 사용할 수 있는 빌드 시스템을 선택하는 것을 강력히 권장합니다. Maven이나 Gradle을 선택하는 것을 추천합니다. 다른 빌드 시스템(예: Ant)으로도 Spring Boot를 사용할 수 있지만, 특별히 잘 지원되지는 않습니다.

## 의존성 관리

Spring Boot의 각 릴리스는 지원하는 의존성들의 큐레이션된 목록을 제공합니다. 실제로 Spring Boot가 관리해주기 때문에 빌드 설정에서 이러한 의존성들의 버전을 지정할 필요가 없습니다. Spring Boot를 업그레이드하면 이러한 의존성들도 일관된 방식으로 함께 업그레이드됩니다.

> 필요한 경우 버전을 지정하여 Spring Boot의 권장 사항을 재정의할 수 있습니다.

이 큐레이션된 목록에는 Spring Boot와 함께 사용할 수 있는 모든 Spring 모듈과 정제된 서드파티 라이브러리 목록이 포함되어 있습니다. 이 목록은 Maven과 Gradle 모두에서 사용할 수 있는 표준 Bills of Materials(`spring-boot-dependencies`)로 제공됩니다.

> Spring Boot의 각 릴리스는 Spring Framework의 기본 버전과 연관되어 있습니다. 버전을 직접 지정하지 않는 것을 **강력히** 권장합니다.

## Maven

Maven과 함께 Spring Boot를 사용하는 방법에 대해 알아보려면 Spring Boot의 Maven 플러그인 문서를 참조하세요:

* [Reference](../../build-tool-plugins/maven.html)
* [API](../../maven-plugin/api.html)

## Gradle

Gradle과 함께 Spring Boot를 사용하는 방법에 대해 알아보려면 Spring Boot의 Gradle 플러그인 문서를 참조하세요:

* [Reference](../../build-tool-plugins/gradle.html)
* [API](../../gradle-plugin/api.html)

## Ant

Apache Ant+Ivy를 사용하여 Spring Boot 프로젝트를 빌드할 수 있습니다. 실행 가능한 jar를 생성하는 데 도움이 되는 `spring-boot-antlib` "AntLib" 모듈도 제공됩니다.

의존성을 선언하기 위한 일반적인 `ivy.xml` 파일은 다음과 같습니다:

```xml
<ivy-module version="2.0">
    <info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
    <configurations>
        <conf name="compile" description="everything needed to compile this module" />
        <conf name="runtime" extends="compile" description="everything needed to run this module" />
    </configurations>
    <dependencies>
        <dependency org="org.springframework.boot" name="spring-boot-starter"
            rev="${spring-boot.version}" conf="compile" />
    </dependencies>
</ivy-module>
```

일반적인 `build.xml`은 다음과 같습니다:

```xml
<project
    xmlns:ivy="antlib:org.apache.ivy.ant"
    xmlns:spring-boot="antlib:org.springframework.boot.ant"
    name="myapp" default="build">

    <property name="spring-boot.version" value="3.4.3" />

    <target name="resolve" description="--> retrieve dependencies with ivy">
        <ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
    </target>

    <target name="classpaths" depends="resolve">
        <path id="compile.classpath">
            <fileset dir="lib/compile" includes="*.jar" />
        </path>
    </target>

    <target name="init" depends="classpaths">
        <mkdir dir="build/classes" />
    </target>

    <target name="compile" depends="init" description="compile">
        <javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
    </target>

    <target name="build" depends="compile">
        <spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
            <spring-boot:lib>
                <fileset dir="lib/runtime" />
            </spring-boot:lib>
        </spring-boot:exejar>
    </target>
</project>
```

> spring-boot-antlib 모듈을 사용하지 않으려면 ["How-to Guides"의 "spring-boot-antlib을 사용하지 않고 Ant로 실행 가능한 아카이브 빌드하기"](../../how-to/build.html#howto.build.build-an-executable-archive-with-ant-without-using-spring-boot-antlib) 섹션을 참조하세요.

## Starters

Starters는 애플리케이션에 포함할 수 있는 편리한 dependency descriptors 집합입니다. 이를 통해 다양한 Spring 기술 및 관련 기술을 신속하게 사용할 수 있으며, 샘플 코드를 뒤지거나 dependency descriptors를 복사하여 붙여넣을 필요가 없습니다. 예를 들어, Spring과 JPA를 사용하여 데이터베이스 접근을 시작하려면 프로젝트에 `spring-boot-starter-data-jpa` 의존성을 포함하면 됩니다.

스타터에는 프로젝트를 신속하게 실행할 수 있도록 필요한 많은 의존성이 포함되어 있으며, 일관적이고 지원되는 관리된 transitive dependencies을 제공합니다.

> 공식 스타터는 `spring-boot-starter-*` 명명 패턴을 따르는 것이 매우 중요합니다. 이는 사용자가 스타터를 쉽게 찾을 수 있게 해줍니다. 서드파티 스타터는 프로젝트 이름을 스타터 이름의 접두사로 사용해야 합니다. 예를 들어, 서드파티 프로젝트 "thirdpartyproject"의 스타터는 `thirdpartyproject-spring-boot-starter`로 이름을 지정해야 합니다.

다음 애플리케이션 스타터는 Spring Boot에서 `org.springframework.boot` 그룹 하에 제공됩니다:

| Name | Description |
|------|-------------|
| `spring-boot-starter` | 코어 스타터, auto-configuration 지원, logging, YAML 포함 |
| `spring-boot-starter-amqp` | Spring AMQP와 RabbitMQ 사용을 위한 스타터 |
| `spring-boot-starter-aop` | Spring AOP와 AspectJ를 사용한 aspect 지향 프로그래밍 지원 |
| `spring-boot-starter-artemis` | Apache Artemis를 사용한 JMS 메시징 지원 |
| `spring-boot-starter-batch` | Spring Batch 지원 |
| `spring-boot-starter-cache` | Spring Framework의 캐싱 지원 |
| `spring-boot-starter-data-cassandra` | Cassandra 분산 데이터베이스와 Spring Data Cassandra 지원 |
| `spring-boot-starter-data-cassandra-reactive` | Cassandra 분산 데이터베이스와 Spring Data Cassandra Reactive 지원 |
| `spring-boot-starter-data-couchbase` | Couchbase 문서 지향 데이터베이스와 Spring Data Couchbase 지원 |
| `spring-boot-starter-data-couchbase-reactive` | Couchbase 문서 지향 데이터베이스와 Spring Data Couchbase Reactive 지원 |
| `spring-boot-starter-data-elasticsearch` | Elasticsearch 검색 및 분석 엔진과 Spring Data Elasticsearch 지원 |
| `spring-boot-starter-data-jdbc` | Spring Data JDBC 지원 |
| `spring-boot-starter-data-jpa` | Spring Data JPA와 Hibernate 지원 |
| `spring-boot-starter-data-ldap` | Spring Data LDAP 지원 |
| `spring-boot-starter-data-mongodb` | MongoDB 문서 지향 데이터베이스와 Spring Data MongoDB 지원 |
| `spring-boot-starter-data-mongodb-reactive` | MongoDB 문서 지향 데이터베이스와 Spring Data MongoDB Reactive 지원 |
| `spring-boot-starter-data-neo4j` | Neo4j 그래프 데이터베이스와 Spring Data Neo4j 지원 |
| `spring-boot-starter-data-r2dbc` | Spring Data R2DBC 지원 |
| `spring-boot-starter-data-redis` | Redis 키-값 데이터 저장소와 Spring Data Redis, Lettuce 클라이언트 지원 |
| `spring-boot-starter-data-redis-reactive` | Redis 키-값 데이터 저장소와 Spring Data Redis Reactive, Lettuce 클라이언트 지원 |
| `spring-boot-starter-data-rest` | Spring Data REST를 사용하여 REST 서비스를 통해 Spring Data 저장소를 노출하기 위한 스타터 |
| `spring-boot-starter-data-solr` | Apache Solr 검색 플랫폼과 Spring Data Solr 지원 |
| `spring-boot-starter-freemarker` | FreeMarker 뷰를 사용한 MVC 웹 애플리케이션 구축 지원 |
| `spring-boot-starter-graphql` | Spring GraphQL 지원 |
| `spring-boot-starter-groovy-templates` | Groovy 템플릿 뷰를 사용한 MVC 웹 애플리케이션 구축 지원 |
| `spring-boot-starter-hateoas` | Spring MVC와 Spring HATEOAS를 사용한 하이퍼미디어 기반 RESTful 웹 애플리케이션 구축 지원 |
| `spring-boot-starter-integration` | Spring Integration 지원 |
| `spring-boot-starter-jdbc` | JDBC와 HikariCP 커넥션 풀 지원 |
| `spring-boot-starter-jersey` | JAX-RS와 Jersey를 사용한 RESTful 웹 애플리케이션 구축을 위한 대안. `spring-boot-starter-web` 참고 |
| `spring-boot-starter-jooq` | jOOQ를 사용한 SQL 데이터베이스 접근 지원. `spring-boot-starter-data-jpa` 또는 `spring-boot-starter-jdbc`의 대안 |
| `spring-boot-starter-json` | JSON 읽기 및 쓰기 지원 |
| `spring-boot-starter-mail` | Java Mail과 Spring Framework의 이메일 전송 지원 |
| `spring-boot-starter-mustache` | Mustache 뷰를 사용한 MVC 웹 애플리케이션 구축 지원 |
| `spring-boot-starter-oauth2-client` | Spring Security의 OAuth2/OpenID Connect 클라이언트 기능 지원 |
| `spring-boot-starter-oauth2-resource-server` | Spring Security의 OAuth2 리소스 서버 기능 지원 |
| `spring-boot-starter-quartz` | Quartz 스케줄러 지원 |
| `spring-boot-starter-rsocket` | RSocket 클라이언트와 서버 구축 지원 |
| `spring-boot-starter-security` | Spring Security 지원 |
| `spring-boot-starter-test` | JUnit Jupiter, Hamcrest, Mockito를 포함한 라이브러리로 Spring Boot 애플리케이션 테스트 지원 |
| `spring-boot-starter-thymeleaf` | Thymeleaf 뷰를 사용한 MVC 웹 애플리케이션 구축 지원 |
| `spring-boot-starter-validation` | Java Bean Validation과 Hibernate Validator 지원 |
| `spring-boot-starter-web` | Spring MVC를 사용한 RESTful 애플리케이션을 포함한 웹 애플리케이션 구축 지원. Tomcat을 기본 내장 컨테이너로 사용 |
| `spring-boot-starter-web-services` | Spring Web Services 지원 |
| `spring-boot-starter-webflux` | Spring Framework의 Reactive Web 지원을 사용한 WebFlux 애플리케이션 구축 지원 |
| `spring-boot-starter-websocket` | Spring Framework의 WebSocket 지원을 사용한 WebSocket 애플리케이션 구축 지원 |

프로덕션 환경에서 유용한 다음의 스타터들도 제공됩니다:

| Name | Description |
|------|-------------|
| `spring-boot-starter-actuator` | Spring Boot Actuator를 사용한 프로덕션 준비 모니터링과 관리 기능 지원 |

Spring Boot는 특정 기술적 측면을 제외하거나 교환하고 싶을 때 사용할 수 있는 여러 스타터를 포함하고 있습니다.
| Name | Description |
|------|-------------|
| `spring-boot-starter-jetty` | Jetty를 내장 서블릿 컨테이너로 사용. `spring-boot-starter-tomcat`의 대안 |
| `spring-boot-starter-log4j2` | Log4j2를 로깅에 사용. `spring-boot-starter-logging`의 대안 |
| `spring-boot-starter-logging` | Logback을 사용한 로깅 지원. 기본 로깅 스타터 |
| `spring-boot-starter-reactor-netty` | Reactor Netty를 내장 리액티브 HTTP 서버로 사용 |
| `spring-boot-starter-tomcat` | Tomcat을 내장 서블릿 컨테이너로 사용. `spring-boot-starter-web`의 기본 서블릿 컨테이너 |
| `spring-boot-starter-undertow` | Undertow를 내장 서블릿 컨테이너로 사용. `spring-boot-starter-tomcat`의 대안 |

기술적 측면을 교환하는 방법을 배우려면 웹 서버 및 로깅 시스템 교환에 대한 설명서를 참조하십시오.

> 💡 공식 스타터 목록에 대한 자세한 내용은 [GitHub](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-starters/README.adoc)의 `spring-boot-starters` 모듈에서 확인할 수 있습니다.