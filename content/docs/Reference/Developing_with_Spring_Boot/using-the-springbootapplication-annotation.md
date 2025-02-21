---
title: Using the @SpringBootApplication Annotation
date: 2025-02-21
weight: 7
---

## @SpringBootApplication 어노테이션 사용

많은 Spring Boot 개발자들은 자신의 앱에서 자동 구성, 컴포넌트 스캔을 사용하고 "애플리케이션 클래스"에서 추가 구성을 정의하는 것을 선호합니다. 단일 [@SpringBootApplication](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html   ) 어노테이션을 사용하여 이 세 가지 기능을 활성화할 수 있습니다:

- [@EnableAutoConfiguration](https://docs.spring.io/spring-boot/docs/3.4.3/api/org/springframework/boot/autoconfigure/EnableAutoConfiguration.html): Spring Boot의 자동 구성 메커니즘 활성화

- [@ComponentScan](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/ComponentScan.html): 애플리케이션이 위치한 패키지에서 @Component 스캔 활성화 (모범 사례 참조)

- [@SpringBootConfiguration](https://docs.spring.io/spring-boot/docs/3.4.3/api/org/springframework/boot/SpringBootConfiguration.html): 컨텍스트에서 추가 빈 등록이나 추가 구성 클래스의 가져오기를 활성화. 통합 테스트에서 구성 감지를 돕는 Spring의 표준 @Configuration의 대안입니다.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// @SpringBootConfiguration @EnableAutoConfiguration @ComponentScan과 동일
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

@SpringBootApplication은 또한 @EnableAutoConfiguration과 @ComponentScan의 속성을 커스터마이즈하기 위한 별칭을 제공합니다.

> **💡 참고**
> 
> 이러한 기능들은 필수가 아니며, 이 단일 어노테이션을 그것이 활성화하는 기능들 중 어느 것으로든 대체할 수 있습니다. 예를 들어, 애플리케이션에서 @ComponentScan이나 @EnableAutoConfiguration을 사용하지 않을 수 있습니다:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.Import;

@SpringBootConfiguration(proxyBeanMethods = false)
@EnableAutoConfiguration
@Import({ SomeConfiguration.class, AnotherConfiguration.class })
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

이 예제에서 MyApplication은 다른 Spring Boot 애플리케이션과 동일하지만, @Component가 달린 클래스와 @ConfigurationProperties가 달린 클래스가 자동으로 감지되지 않고 사용자 정의 빈들이 명시적으로 가져와집니다([@Import](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/Import.html) 참조).

---