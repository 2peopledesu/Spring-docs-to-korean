---
title: Running Your Application
date: 2024-02-21
weight: 8
---

## 애플리케이션 실행

애플리케이션을 jar로 패키징하고 내장 HTTP 서버를 사용하는 가장 큰 장점 중 하나는 다른 일반적인 애플리케이션처럼 실행할 수 있다는 것입니다. Spring Boot 애플리케이션을 디버깅할 때도 마찬가지입니다. 특별한 IDE 플러그인이나 확장 기능이 필요하지 않습니다.

> **💡 참고**
> 
> 아래 옵션들은 개발을 위해 로컬에서 애플리케이션을 실행하는 데 가장 적합합니다. 프로덕션 배포는 [프로덕션용 애플리케이션 패키징](../packaging-for-production)을 참조하세요.

> **💡 참고**
> 
> 이 섹션은 jar 기반 패키징만 다룹니다. war 파일로 패키징하기로 선택한 경우 서버와 IDE 문서를 참조하세요.

## IDE에서 실행하기

Spring Boot 애플리케이션을 Java 애플리케이션으로 IDE에서 실행할 수 있습니다. 하지만 먼저 프로젝트를 가져와야 합니다. 가져오기 단계는 IDE와 빌드 시스템에 따라 다릅니다. 대부분의 IDE는 Maven 프로젝트를 직접 가져올 수 있습니다. 예를 들어, Eclipse 사용자는 `File` 메뉴에서 `Import…​` → `Existing Maven Projects`를 선택할 수 있습니다.

IDE에 프로젝트를 직접 가져올 수 없는 경우, 빌드 플러그인을 사용하여 IDE 메타데이터를 생성할 수 있습니다. Maven은 [Eclipse](https://maven.apache.org/plugins/maven-eclipse-plugin/)와 [IDEA](https://maven.apache.org/plugins/maven-idea-plugin/)용 플러그인을 포함합니다. Gradle은 [다양한 IDE](https://docs.gradle.org/current/userguide/userguide.html)용 플러그인을 제공합니다.

> **⚠️ 주의**
> 
> 웹 애플리케이션을 실수로 두 번 실행하면 "Port already in use" 오류가 표시됩니다. Spring Tools 사용자는 기존 인스턴스가 종료되도록 Run 버튼 대신 Relaunch 버튼을 사용할 수 있습니다.

## 패키지된 애플리케이션으로 실행하기

Spring Boot Maven 또는 Gradle 플러그인을 사용하여 실행 가능한 jar를 만든 경우, 다음 예제와 같이 `java -jar`를 사용하여 애플리케이션을 실행할 수 있습니다:

```shell
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

패키지된 애플리케이션을 원격 디버깅 지원이 활성화된 상태로 실행할 수도 있습니다. 이를 통해 다음 예제와 같이 패키지된 애플리케이션에 디버거를 연결할 수 있습니다:

```shell
$ java -agentlib:jdwp=server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

## Maven 플러그인 사용하기

Spring Boot Maven 플러그인은 애플리케이션을 빠르게 컴파일하고 실행하는 데 사용할 수 있는 `run` 을 목표로합니다. 애플리케이션은 IDE에서처럼 압축이 풀린 형태로 실행됩니다. 다음 예제는 Spring Boot 애플리케이션을 실행하는 일반적인 Maven 명령을 보여줍니다:

```shell
$ mvn spring-boot:run
```

다음 예제와 같이 `MAVEN_OPTS` 운영 체제 환경 변수를 사용할 수도 있습니다:

```shell
$ export MAVEN_OPTS=-Xmx1024m
```

## Gradle 플러그인 사용하기

Spring Boot Gradle 플러그인도 애플리케이션을 압축이 풀린 형태로 실행하는 데 사용할 수 있는 `bootRun` 태스크를 포함합니다. `bootRun` 태스크는 `org.springframework.boot`와 `java` 플러그인을 적용할 때마다 추가되며 다음 예제와 같이 사용됩니다:

```shell
$ gradle bootRun
```

다음 예제와 같이 `JAVA_OPTS` 운영 체제 환경 변수를 사용할 수도 있습니다:

```shell
$ export JAVA_OPTS=-Xmx1024m
```

## Hot Swapping

Spring Boot 애플리케이션은 일반 Java 애플리케이션이므로 JVM Hot Swapping이 기본적으로 작동해야 합니다. JVM Hot Swapping은 교체할 수 있는 바이트코드에 다소 제한이 있습니다. 더 완벽한 솔루션을 위해서는 [JRebel](https://www.jrebel.com/products/jrebel)을 사용할 수 있습니다.

`spring-boot-devtools` 모듈은 또한 빠른 애플리케이션 재시작을 지원합니다. 자세한 내용은 "How-to Guides"의 [Hot Swapping](../../../how-to/hotswapping.html) 섹션을 참조하세요.

---