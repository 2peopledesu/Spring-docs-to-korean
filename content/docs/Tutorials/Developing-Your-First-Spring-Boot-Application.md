---
title: Developing Your First Spring Boot Application
date: 2024-02-21
weight: 1
---

## 첫 번째 Spring Boot 애플리케이션 개발하기

이 섹션에서는 Spring Boot의 주요 기능을 강조하는 간단한 "Hello World!" 웹 애플리케이션을 개발하는 방법을 설명합니다. 빌드 시스템으로 Maven 또는 Gradle 중에서 선택할 수 있습니다.

> [spring.io](https://spring.io) 웹사이트에는 Spring Boot를 사용하는 여러 "Getting Started" [가이드](https://spring.io/guides)가 포함되어 있습니다. 특정 문제를 해결해야 하는 경우, 먼저 그곳을 확인하세요.
>
> 아래 단계는 [start.spring.io](https://start.spring.io)로 이동하여 종속성 검색기에서 "Web" Starter를 선택함으로써 단축할 수 있습니다. 이렇게 하면 새로운 프로젝트 구조가 생성되어 바로 코딩을 시작할 수 있습니다.

## 전제 조건

시작하기 전에 터미널을 열고 다음 명령어를 실행하여 유효한 버전의 Java가 설치되어 있는지 확인합니다:

```bash
$ java -version
openjdk version "17.0.4.1" 2022-08-12 LTS
OpenJDK Runtime Environment (build 17.0.4.1+1-LTS)
OpenJDK 64-Bit Server VM (build 17.0.4.1+1-LTS, mixed mode, sharing)
```

이 샘플은 자체 디렉토리에서 생성되어야 합니다. 이후 지침에서는 적절한 디렉토리를 생성하고 현재 디렉토리로 설정했다고 가정합니다.

### Maven 설치 확인

Maven을 사용하려면 Maven이 설치되어 있는지 확인합니다:

```bash
$ mvn -v
Apache Maven 3.8.5 (3599d3414f046de2324203b78ddcf9b5e4388aa0)
Maven home: usr/Users/developer/tools/maven/3.8.5
Java version: 17.0.4.1, vendor: BellSoft, runtime: /Users/developer/sdkman/candidates/java/17.0.4.1-librca
```

### Gradle 설치 확인

Gradle을 사용하려면 Gradle이 설치되어 있는지 확인합니다:

```bash
$ gradle --version

------------------------------------------------------------
Gradle 8.1.1
------------------------------------------------------------

Build time:   2023-04-21 12:31:26 UTC
Revision:     1cf537a851c635c364a4214885f8b9798051175b

Kotlin:       1.8.10
Groovy:       3.0.15
Ant:          Apache Ant(TM) version 1.10.11 compiled on July 10 2021
JVM:          17.0.7 (BellSoft 17.0.7+7-LTS)
OS:           Linux 6.2.12-200.fc37.aarch64 aarch64
```

## Maven을 사용한 프로젝트 설정

Maven <code>pom.xml</code> 파일을 생성하는 것으로 시작해야 합니다. <code>pom.xml</code>은 프로젝트를 빌드하는 데 사용되는 레시피입니다. 좋아하는 텍스트 편집기를 열고 다음 내용을 추가합니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.3</version>
    </parent>

    <!-- Additional lines to be added here... -->

</project>
```

위의 목록은 작동하는 빌드를 제공해야 합니다. 이제 <code>mvn package</code>를 실행하여 테스트할 수 있습니다(현재는 "jar will be empty - no content was marked for inclusion!" 경고를 무시해도 됩니다).

> 이 시점에서 프로젝트를 IDE에 가져올 수 있습니다(대부분의 현대 Java IDE는 Maven에 대한 기본 지원을 포함하고 있습니다). 단순함을 위해, 이 예에서는 일반 텍스트 편집기를 계속 사용합니다.

## Gradle을 사용한 프로젝트 설정

Gradle `build.gradle` 파일을 생성하는 것으로 시작해야 합니다. `build.gradle`은 프로젝트를 빌드하는 데 사용되는 빌드 스크립트입니다. 좋아하는 텍스트 편집기를 열고 다음 내용을 추가합니다:

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.4.3'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
}
```

위의 목록은 작동하는 빌드를 제공해야 합니다. `gradle classes` 명령어를 실행하여 테스트할 수 있습니다.

> 이 시점에서 프로젝트를 IDE에 가져올 수 있습니다(대부분의 현대 Java IDE는 Gradle에 대한 기본 지원을 포함하고 있습니다). 단순함을 위해, 이 예에서는 일반 텍스트 편집기를 계속 사용합니다.

## Classpath 의존성 추가

Spring Boot는 Classpath에 JAR 파일을 추가할 수 있는 여러 "Starter"를 제공합니다. Starter는 특정 유형의 애플리케이션을 개발할 때 필요할 가능성이 있는 의존성을 제공합니다.

### Maven 의존성

대부분의 Spring Boot 애플리케이션은 POM의 부모 섹션에서 `spring-boot-starter-parent`를 사용합니다. `spring-boot-starter-parent`는 유용한 Maven 기본값을 제공하는 특별한 Starter입니다. 또한 "blessed" 의존성에 대해 버전 태그를 생략할 수 있도록 의존성 관리 섹션을 제공합니다.

웹 애플리케이션을 개발하고 있으므로 `spring-boot-starter-web` 의존성을 추가합니다. 그 전에 현재 가지고 있는 의존성을 확인하기 위해 다음 명령어를 실행합니다:

```bash
$ mvn dependency:tree

[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```

`mvn dependency:tree` 명령어는 프로젝트 의존성의 트리 표현을 출력합니다. `spring-boot-starter-parent`는 자체적으로 의존성을 제공하지 않음을 알 수 있습니다. 필요한 의존성을 추가하기 위해 `pom.xml`을 편집하고 부모 섹션 바로 아래에 `spring-boot-starter-web` 의존성을 추가합니다:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

`mvn dependency:tree`를 다시 실행하면 Tomcat 웹 서버와 Spring Boot 자체를 포함한 여러 추가 의존성이 나타나는 것을 볼 수 있습니다.

### Gradle 의존성

대부분의 Spring Boot 애플리케이션은 `org.springframework.boot` Gradle 플러그인을 사용합니다. 이 플러그인은 유용한 기본값과 Gradle 태스크를 제공합니다. `io.spring.dependency-management` Gradle 플러그인은 "blessed" 의존성에 대해 버전 태그를 생략할 수 있도록 의존성 관리를 제공합니다.

웹 애플리케이션을 개발하고 있으므로 `spring-boot-starter-web` 의존성을 추가합니다. 그 전에 현재 가지고 있는 의존성을 확인하기 위해 다음 명령어를 실행합니다:

```bash
$ gradle dependencies

> Task :dependencies

------------------------------------------------------------
Root project 'myproject'
------------------------------------------------------------
```

`gradle dependencies` 명령어는 프로젝트 의존성의 트리 표현을 출력합니다. 현재 프로젝트에는 의존성이 없습니다. 필요한 의존성을 추가하기 위해 `build.gradle`을 편집하고 dependencies 섹션에 `spring-boot-starter-web` 의존성을 추가합니다:

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

`gradle dependencies`를 다시 실행하면 Tomcat 웹 서버와 Spring Boot 자체를 포함한 여러 추가 의존성이 나타나는 것을 볼 수 있습니다.

## 코드 작성하기

애플리케이션을 완성하기 위해 하나의 Java 파일을 생성해야 합니다. 기본적으로 Maven과 Gradle은 `src/main/java`에서 소스를 컴파일하므로, 해당 디렉토리 구조를 생성하고 `src/main/java/com/example/MyApplication.java` 파일을 추가하여 다음 코드를 포함시켜야 합니다:

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class MyApplication {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

코드가 많지 않지만 많은 일이 일어나고 있습니다. 다음 섹션에서 중요한 부분들을 살펴보겠습니다.

### @RestController와 @RequestMapping 어노테이션

`MyApplication` 클래스의 첫 번째 어노테이션은 `@RestController`입니다. 이것은 stereotype 어노테이션으로 알려져 있습니다. 코드를 읽는 사람들과 Spring에게 클래스가 특정 역할을 수행한다는 힌트를 제공합니다. 이 경우 우리의 클래스는 웹 `@Controller`이므로, Spring은 들어오는 웹 요청을 처리할 때 이를 고려합니다.

`@RequestMapping` 어노테이션은 "라우팅" 정보를 제공합니다. `/` 경로를 가진 모든 HTTP 요청이 `home` 메서드에 매핑되어야 한다고 Spring에 알려줍니다. `@RestController` 어노테이션은 결과 문자열을 직접 호출자에게 반환하도록 Spring에 지시합니다.

> `@RestController`와 `@RequestMapping` 어노테이션은 Spring MVC 어노테이션입니다(Spring Boot에만 특화된 것이 아닙니다). 자세한 내용은 Spring 레퍼런스 문서의 MVC 섹션을 참조하세요.

### @SpringBootApplication 어노테이션

두 번째 클래스 레벨 어노테이션은 `@SpringBootApplication`입니다. 이 어노테이션은 meta-annotation으로 알려져 있으며, `@SpringBootConfiguration`, `@EnableAutoConfiguration`, `@ComponentScan`을 결합합니다.

이 중에서 여기서 가장 관심 있는 어노테이션은 `@EnableAutoConfiguration`입니다. `@EnableAutoConfiguration`은 추가한 jar 의존성을 기반으로 Spring을 어떻게 구성하고 싶은지 "추측"하도록 Spring Boot에 지시합니다. `spring-boot-starter-web`이 Tomcat과 Spring MVC를 추가했기 때문에, 자동 구성은 웹 애플리케이션을 개발하고 있다고 가정하고 그에 따라 Spring을 설정합니다.

> **Starter와 자동 구성**
>
> 자동 구성은 Starter와 잘 작동하도록 설계되었지만, 두 개념이 직접적으로 연결되어 있지는 않습니다. Starter 외부의 jar 의존성을 자유롭게 선택할 수 있으며, Spring Boot는 여전히 애플리케이션을 자동 구성하기 위해 최선을 다합니다.

### "main" 메서드

애플리케이션의 마지막 부분은 `main` 메서드입니다. 이는 Java 관례를 따르는 애플리케이션 진입점을 위한 표준 메서드입니다. 우리의 `main` 메서드는 `run`을 호출하여 Spring Boot의 `SpringApplication` 클래스에 위임합니다. `SpringApplication`은 우리의 애플리케이션을 부트스트랩하고 Spring을 시작하며, 이는 차례로 자동 구성된 Tomcat 웹 서버를 시작합니다. 어떤 것이 주요 Spring 컴포넌트인지 `SpringApplication`에 알려주기 위해 `MyApplication.class`를 `run` 메서드의 인자로 전달해야 합니다. `args` 배열도 명령줄 인자를 노출하기 위해 전달됩니다.

## 예제 실행하기

### Maven

이 시점에서 애플리케이션이 작동해야 합니다. `spring-boot-starter-parent` POM을 사용했기 때문에 애플리케이션을 시작하는 데 사용할 수 있는 유용한 `run` 목표가 있습니다. 루트 프로젝트 디렉토리에서 `mvn spring-boot:run`을 입력하여 애플리케이션을 시작합니다. 다음과 유사한 출력이 표시되어야 합니다:

```bash
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v3.4.3)
....... . . .
....... . . . (log output here)
....... . . .
........ Started MyApplication in 0.906 seconds (process running for 6.514)
```

웹 브라우저에서 `localhost:8080`을 열면 다음과 같은 출력이 표시되어야 합니다:

```
Hello World!
```

애플리케이션을 정상적으로 종료하려면 `ctrl-c`를 누르세요.

### Gradle

이 시점에서 애플리케이션이 작동해야 합니다. `org.springframework.boot` Gradle 플러그인을 사용했기 때문에 애플리케이션을 시작하는 데 사용할 수 있는 유용한 `bootRun` 목표가 있습니다. 루트 프로젝트 디렉토리에서 `gradle bootRun`을 입력하여 애플리케이션을 시작합니다. 다음과 유사한 출력이 표시되어야 합니다:

```bash
$ gradle bootRun

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v3.4.3)
....... . . .
....... . . . (log output here)
....... . . .
........ Started MyApplication in 0.906 seconds (process running for 6.514)
```

웹 브라우저에서 `localhost:8080`을 열면 다음과 같은 출력이 표시되어야 합니다:

```
Hello World!
```

애플리케이션을 정상적으로 종료하려면 `ctrl-c`를 누르세요.

## 실행 가능한 Jar 생성하기

프로덕션에서 실행할 수 있는 완전히 독립적인 실행 가능한 jar 파일을 생성하여 예제를 마무리하겠습니다. 실행 가능한 jar("uber jar" 또는 "fat jar"라고도 함)는 컴파일된 클래스와 코드 실행에 필요한 모든 jar 의존성을 포함하는 아카이브입니다.

> **실행 가능한 jar와 Java**
>
> Java는 중첩된 jar 파일(jar 내부에 포함된 jar 파일)을 로드하는 표준 방법을 제공하지 않습니다. 이는 독립적인 애플리케이션을 배포하려는 경우 문제가 될 수 있습니다.
>
> 이 문제를 해결하기 위해 많은 개발자들이 "uber" jar를 사용합니다. uber jar는 애플리케이션의 모든 의존성의 모든 클래스를 단일 아카이브로 패키징합니다. 이 접근 방식의 문제는 애플리케이션에 어떤 라이브러리가 있는지 보기 어렵다는 것입니다. 또한 여러 jar에서 동일한 파일 이름(다른 내용)이 사용되는 경우 문제가 될 수 있습니다.
>
> Spring Boot는 다른 접근 방식을 취하고 실제로 jar를 직접 중첩할 수 있도록 합니다.

### Maven

실행 가능한 jar를 생성하기 위해 `spring-boot-maven-plugin`을 `pom.xml`에 추가해야 합니다. 이를 위해 다음 줄을 dependencies 섹션 바로 아래에 삽입합니다:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

> `spring-boot-starter-parent` POM은 repackage 목표를 바인딩하기 위한 `<executions>` 구성을 포함합니다. 부모 POM을 사용하지 않는 경우 이 구성을 직접 선언해야 합니다. 자세한 내용은 [플러그인 문서](../../maven-plugin/getting-started.html)를 참조하세요.

`pom.xml`을 저장하고 다음과 같이 명령줄에서 `mvn package`를 실행합니다:

```bash
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:3.4.3:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

`target` 디렉토리를 보면 `myproject-0.0.1-SNAPSHOT.jar`가 보일 것입니다. 파일 크기는 약 18MB여야 합니다. 내부를 살펴보려면 다음과 같이 `jar tvf`를 사용할 수 있습니다:

```bash
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```

`target` 디렉토리에서 `myproject-0.0.1-SNAPSHOT.jar.original`이라는 훨씬 작은 파일도 볼 수 있습니다. 이는 Spring Boot가 재패키징하기 전에 Maven이 생성한 원본 jar 파일입니다.

애플리케이션을 실행하려면 다음과 같이 `java -jar` 명령을 사용합니다:

```bash
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v3.4.3)
....... . . .
....... . . . (log output here)
....... . . .
........ Started MyApplication in 0.999 seconds (process running for 1.253)
```

이전과 마찬가지로 애플리케이션을 종료하려면 `ctrl-c`를 누르세요.

### Gradle

실행 가능한 jar를 생성하기 위해 다음과 같이 명령줄에서 `gradle bootJar`를 실행합니다:

```bash
$ gradle bootJar

BUILD SUCCESSFUL in 639ms
3 actionable tasks: 3 executed
```

`build/libs` 디렉토리를 보면 `myproject-0.0.1-SNAPSHOT.jar`가 보일 것입니다. 파일 크기는 약 18MB여야 합니다. 내부를 살펴보려면 다음과 같이 `jar tvf`를 사용할 수 있습니다:

```bash
$ jar tvf build/libs/myproject-0.0.1-SNAPSHOT.jar
```

애플리케이션을 실행하려면 다음과 같이 `java -jar` 명령을 사용합니다:

```bash
$ java -jar build/libs/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v3.4.3)
....... . . .
....... . . . (log output here)
....... . . .
........ Started MyApplication in 0.999 seconds (process running for 1.253)
```

이전과 마찬가지로 애플리케이션을 종료하려면 `ctrl-c`를 누르세요.