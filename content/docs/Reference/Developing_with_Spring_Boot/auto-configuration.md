---
title: Auto-configuration
date: 2025-02-21
weight: 5
---

> **⚠️ 경고**
> 
> 이 버전은 아직 개발 중이며 안정적이지 않습니다. 최신 안정 버전은 [Spring Boot 3.4.3](https://docs.spring.io/spring-boot/docs/3.4.3/reference/html/using.html#using.auto-configuration)을 사용하세요!

## Auto-configuration

Spring Boot 자동 구성은 추가한 jar 의존성을 기반으로 Spring 애플리케이션을 자동으로 구성하려고 시도합니다. 예를 들어, `HSQLDB`가 클래스패스에 있고 데이터베이스 연결 빈을 수동으로 구성하지 않은 경우, Spring Boot는 인메모리 데이터베이스를 자동으로 구성합니다.

자동 구성을 사용하려면 @Configuration 클래스 중 하나에 [@EnableAutoConfiguration](https://docs.spring.io/spring-boot/3.4-SNAPSHOT/api/java/org/springframework/boot/autoconfigure/EnableAutoConfiguration.html) 또는 [@SpringBootApplication](https://docs.spring.io/spring-boot/3.4-SNAPSHOT/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html) 어노테이션을 추가해야 합니다.


> **💡 참고**
> 
> [@SpringBootApplication](https://docs.spring.io/spring-boot/3.4-SNAPSHOT/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html) 또는 [@EnableAutoConfiguration](https://docs.spring.io/spring-boot/3.4-SNAPSHOT/api/java/org/springframework/boot/autoconfigure/EnableAutoConfiguration.html) 어노테이션은 하나만 추가해야 합니다. 일반적으로 주요 [@Configuration](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/Configuration.html) 클래스에만 둘 중 하나를 추가하는 것을 권장합니다.

## 자동 구성 점진적 대체

자동 구성은 비침투적입니다. 언제든지 자신만의 구성을 정의하여 자동 구성의 특정 부분을 대체할 수 있습니다. 예를 들어, 자체 [DataSource](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/javax/sql/DataSource.html) 빈을 추가하면 기본 내장 데이터베이스 지원이 제외됩니다.

현재 어떤 자동 구성이 적용되고 있는지, 그 이유를 알아보려면 `--debug` 스위치를 사용하여 애플리케이션을 시작하세요. 이렇게 하면 핵심 로거들에 대한 디버그 로그가 활성화되고 조건 보고서가 콘솔에 출력됩니다.

## 특정 자동 구성 클래스 비활성화

원하지 않는 특정 자동 구성 클래스가 적용되고 있다면, 다음 예제와 같이 @SpringBootApplication의 exclude 속성을 사용하여 비활성화할 수 있습니다:

Java

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class MyApplication {

}
```

Kotlin

```kotlin
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration

@SpringBootApplication(exclude = [DataSourceAutoConfiguration::class])
class MyApplication
```

클래스가 클래스패스에 없는 경우, 어노테이션의 `excludeName` 속성을 사용하여 정규화된 이름을 대신 지정할 수 있습니다. @SpringBootApplication 대신 @EnableAutoConfiguration을 선호한다면, `exclude`와 `excludeName`도 사용할 수 있습니다. 마지막으로 `spring.autoconfigure.exclude` 프로퍼티를 사용하여 제외할 자동 구성 클래스 목록을 제어할 수도 있습니다.

> **💡 참고**
> 
> 어노테이션 수준과 프로퍼티를 사용하여 모두 제외를 정의할 수 있습니다.

> **⚠️ 주의**
> 
> 자동 구성 클래스가 public이더라도, 클래스의 이름만이 자동 구성을 비활성화하는 데 사용할 수 있는 공개 API로 간주됩니다. 중첩된 구성 클래스나 빈 메서드와 같은 클래스의 실제 내용은 내부용으로만 사용되며, 직접 사용하는 것을 권장하지 않습니다.

## 자동 구성 패키지

자동 구성 패키지는 엔티티나 Spring Data 저장소와 같은 것들을 스캔할 때 기본적으로 살펴보는 패키지입니다. @EnableAutoConfiguration 어노테이션(직접 또는 @SpringBootApplication을 통해)이 기본 자동 구성 패키지를 결정합니다. 추가 패키지는 @AutoConfigurationPackage 어노테이션을 사용하여 구성할 수 있습니다.

---