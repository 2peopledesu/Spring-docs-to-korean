---
title: Profiles
weight: 3
date: 2025-03-14
---

# 프로필

Spring 프로필은 애플리케이션 구성의 일부를 분리하고 특정 환경에서만 사용할 수 있도록 하는 방법을 제공합니다. 다음 예제와 같이 @Component, @Configuration 또는 @ConfigurationProperties는 로드되는 시기를 제한하기 위해 @Profile로 표시될 수 있습니다:

* Java
* Kotlin

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

	// ...

}
```

```kotlin
import org.springframework.context.annotation.Configuration
import org.springframework.context.annotation.Profile

@Configuration(proxyBeanMethods = false)
@Profile("production")
class ProductionConfiguration {

	// ...

}
```

> @ConfigurationProperties 빈이 자동 스캐닝 대신 @EnableConfigurationProperties를 통해 등록되는 경우, @Profile 애노테이션은 @EnableConfigurationProperties 애노테이션이 있는 @Configuration 클래스에 지정되어야 합니다. @ConfigurationProperties가 스캔되는 경우, @Profile은 @ConfigurationProperties 클래스 자체에 지정할 수 있습니다.

어떤 프로필이 활성화되어 있는지 지정하기 위해 `spring.profiles.active` 환경 속성을 사용할 수 있습니다. 이 장에서 앞서 설명한 방법 중 하나로 속성을 지정할 수 있습니다. 예를 들어, 다음 예제와 같이 `application.properties`에 포함시킬 수 있습니다:

* Properties
* YAML

```properties
spring.profiles.active=dev,hsqldb
```

```yaml
spring:
  profiles:
    active: "dev,hsqldb"
```

다음 스위치를 사용하여 명령줄에서도 지정할 수 있습니다: `--spring.profiles.active=dev,hsqldb`.

활성화된 프로필이 없으면 기본 프로필이 활성화됩니다. 기본 프로필의 이름은 `default`이며 다음 예제와 같이 `spring.profiles.default` 환경 속성을 사용하여 조정할 수 있습니다:

* Properties
* YAML

```properties
spring.profiles.default=none
```

```yaml
spring:
  profiles:
    default: "none"
```

`spring.profiles.active`와 `spring.profiles.default`는 프로필이 지정되지 않은 문서에서만 사용할 수 있습니다. 이는 프로필별 파일이나 `spring.config.activate.on-profile`에 의해 활성화된 문서에 포함될 수 없음을 의미합니다.

예를 들어, 두 번째 문서 구성은 유효하지 않습니다:

* Properties
* YAML

```properties
spring.profiles.active=prod
#---
spring.config.activate.on-profile=prod
spring.profiles.active=metrics
```

```yaml
# 이 문서는 유효합니다
spring:
  profiles:
    active: "prod"
---
# 이 문서는 유효하지 않습니다
spring:
  config:
    activate:
      on-profile: "prod"
  profiles:
    active: "metrics"
```

## 활성 프로필 추가하기

`spring.profiles.active` 속성은 다른 속성과 동일한 순서 규칙을 따릅니다: 가장 높은 PropertySource가 우선합니다. 이는 `application.properties`에서 활성 프로필을 지정한 다음 명령줄 스위치를 사용하여 **대체**할 수 있음을 의미합니다.

때로는 활성 프로필을 대체하는 것이 아니라 **추가**하는 속성이 유용할 수 있습니다. `spring.profiles.include` 속성은 `spring.profiles.active` 속성에 의해 활성화된 프로필 위에 활성 프로필을 추가하는 데 사용할 수 있습니다. SpringApplication 진입점에는 추가 프로필을 설정하기 위한 Java API도 있습니다. SpringApplication의 `setAdditionalProfiles()` 메서드를 참조하세요.

예를 들어, 다음 속성이 있는 애플리케이션이 실행될 때, `--spring.profiles.active` 스위치를 사용하여 실행하더라도 common 및 local 프로필이 활성화됩니다:

* Properties
* YAML

```properties
spring.profiles.include[0]=common
spring.profiles.include[1]=local
```

```yaml
spring:
  profiles:
    include:
      - "common"
      - "local"
```

> spring.profiles.active와 유사하게, spring.profiles.include는 프로필이 지정되지 않은 문서에서만 사용할 수 있습니다. 이는 [프로필별 파일](https://docs.spring.io/spring-boot/reference/features/external-config.html#features.external-config.files.profile-specific)이나 spring.config.activate.on-profile에 의해 [활성화된 문서](https://docs.spring.io/spring-boot/reference/features/external-config.html#features.external-config.files.activation-properties)에 포함될 수 없음을 의미합니다.

다음 섹션에서 설명하는 프로필 그룹도 주어진 프로필이 활성화된 경우 활성 프로필을 추가하는 데 사용할 수 있습니다.

## 프로필 그룹

때로는 애플리케이션에서 정의하고 사용하는 프로필이 너무 세분화되어 사용하기 번거로울 수 있습니다. 예를 들어, 데이터베이스 및 메시징 기능을 독립적으로 활성화하기 위해 `proddb` 및 `prodmq` 프로필이 있을 수 있습니다.

이를 돕기 위해 Spring Boot는 프로필 그룹을 정의할 수 있게 합니다. 프로필 그룹을 사용하면 관련 프로필 그룹에 대한 논리적 이름을 정의할 수 있습니다.

예를 들어, `proddb` 및 `prodmq` 프로필로 구성된 `production` 그룹을 만들 수 있습니다.

* Properties
* YAML

```properties
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq
```

```yaml
spring:
  profiles:
    group:
      production:
      - "proddb"
      - "prodmq"
```

이제 애플리케이션은 `--spring.profiles.active=production`을 사용하여 시작하여 한 번에 `production`, `proddb` 및 `prodmq` 프로필을 활성화할 수 있습니다.

> spring.profiles.active 및 spring.profiles.include와 유사하게, spring.profiles.group은 프로필이 지정되지 않은 문서에서만 사용할 수 있습니다. 이는 [프로필별 파일](https://docs.spring.io/spring-boot/reference/features/external-config.html#features.external-config.files.profile-specific)이나 spring.config.activate.on-profile에 의해 [활성화된 문서](https://docs.spring.io/spring-boot/reference/features/external-config.html#features.external-config.files.activation-properties)에 포함될 수 없음을 의미합니다.

## 프로그래밍 방식으로 프로필 설정하기

애플리케이션이 실행되기 전에 `SpringApplication.setAdditionalProfiles(…​)`를 호출하여 프로그래밍 방식으로 활성 프로필을 설정할 수 있습니다. Spring의 ConfigurableEnvironment 인터페이스를 사용하여 프로필을 활성화하는 것도 가능합니다.

## 프로필별 구성 파일

`application.properties`(또는 `application.yaml`)와 @ConfigurationProperties를 통해 참조되는 파일의 프로필별 변형은 모두 파일로 간주되고 로드됩니다. 자세한 내용은 프로필별 파일을 참조하세요.
