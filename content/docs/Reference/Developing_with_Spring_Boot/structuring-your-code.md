---
title: Structuring Your Code
date: 2025-02-21
weight: 3
---

## 코드 구조화

Spring Boot는 작동하기 위해 특정한 코드 레이아웃을 요구하지 않습니다. 하지만 도움이 되는 몇 가지 모범 사례가 있습니다.

> 도메인 기반의 구조를 적용하고 싶다면 [Spring Modulith](https://spring.io/projects/spring-modulith#overview)를 살펴보세요.

## "default" 패키지 사용

클래스에 `package` 선언이 포함되어 있지 않으면 "default package"에 있는 것으로 간주됩니다. "default package" 사용은 일반적으로 권장되지 않으며 피해야 합니다. [@ComponentScan](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/ComponentScan.html), [@ConfigurationPropertiesScan](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/context/properties/ConfigurationPropertiesScan.html), [@EntityScan](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/domain/EntityScan.html) 또는 [@SpringBootApplication](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html) 어노테이션을 사용하는 Spring Boot 애플리케이션에서 특별한 문제를 일으킬 수 있습니다. 모든 jar의 모든 클래스를 읽기 때문입니다.

> Java의 권장 패키지 명명 규칙을 따르고 역방향 도메인 이름(예: `com.example.project`)을 사용하는 것을 권장합니다.

## 메인 애플리케이션 클래스 위치

일반적으로 메인 애플리케이션 클래스를 다른 클래스들보다 상위의 루트 패키지에 위치시키는 것을 권장합니다. [@SpringBootApplication](https://docs.spring.io/spring-boot/reference/using/using-the-springbootapplication-annotation.html)어노테이션은 주로 메인 클래스에 배치되며, 특정 항목에 대한 기본 "검색 패키지"를 암시적으로 정의합니다. 예를 들어, JPA 애플리케이션을 작성하는 경우 [@SpringBootApplication](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html)이 달린 클래스의 패키지가 [@Entity](https://jakarta.ee/specifications/persistence/3.1/apidocs/jakarta.persistence/jakarta/persistence/entity) 항목을 검색하는 데 사용됩니다. 루트 패키지를 사용하면 컴포넌트 스캔이 프로젝트에만 적용되도록 할 수 있습니다.

> [@SpringBootApplication](https://docs.spring.io/spring-boot/3.4.3/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html)을 사용하지 않으려면, 이것이 가져오는 [@EnableAutoConfiguration](https://docs.spring.io/spring-boot/3.4.3/api/java/org/springframework/boot/autoconfigure/EnableAutoConfiguration.html)과 [@ComponentScan](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/ComponentScan.html) 어노테이션을 대신 사용할 수 있습니다.

다음은 일반적인 레이아웃을 보여줍니다:

```none
com
 +- example
     +- myapplication
         +- MyApplication.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

`MyApplication.java` 파일은 기본적인 [@SpringBootApplication](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html)과 함께 `main` 메소드를 다음과 같이 선언합니다:

Java

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```
Kotlin

```kotlin
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
    runApplication<MyApplication>(*args)
}
```