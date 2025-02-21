---
title: Configuration Classes
date: 2024-02-21
weight: 4
---

> **⚠️ 경고**
> 
> 이 버전은 아직 개발 중이며 안정적이지 않습니다. 최신 안정 버전은 [Spring Boot 3.4.3](https://docs.spring.io/spring-boot/docs/3.4.3/reference/html/using.html#using.configuration-classes)을 사용하세요!

## Configuration Classes

Spring Boot는 Java 기반 설정을 선호합니다. [SpringApplication](https://docs.spring.io/spring-boot/docs/3.4.3/api/org/springframework/boot/SpringApplication.html)에서 XML 소스를 사용할 수 있지만, 일반적으로 단일 [@Configuration](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/Configuration.html) 클래스를 주요 소스로 사용하는 것을 권장합니다. 보통 `main` 메소드를 정의하는 클래스가 주요 @Configuration으로 적합한 후보입니다.

> **💡 참고**
> 
> 인터넷에는 XML 설정을 사용하는 많은 Spring 설정 예제들이 게시되어 있습니다. 가능하다면 항상 동등한 Java 기반 설정을 사용하도록 하세요. `Enable*` 어노테이션을 검색하는 것이 좋은 시작점이 될 수 있습니다.

## 추가 Configuration 클래스 가져오기

모든 @Configuration을 단일 클래스에 넣을 필요는 없습니다. [@Import](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/Import.html) 어노테이션을 사용하여 추가 설정 클래스를 가져올 수 있습니다. 또는 [@ComponentScan](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/ComponentScan.html)을 사용하여 @Configuration 클래스를 포함한 모든 Spring 컴포넌트를 자동으로 감지할 수 있습니다.

## XML 설정 가져오기

XML 기반 설정을 반드시 사용해야 하는 경우에도, @Configuration 클래스로 시작하는 것을 권장합니다. 그런 다음 [@ImportResource](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/ImportResource.html) 어노테이션을 사용하여 XML 설정 파일을 로드할 수 있습니다.

---