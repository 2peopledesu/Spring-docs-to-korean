---
title: Externalized Configuration
weight: 2
date: 2025-03-14
---

# 외부화된 구성

Spring Boot는 구성 파일을 외부화하여 동일한 애플리케이션 코드를 다양한 환경에서 사용할 수 있도록 합니다. Java properties 파일, YAML 파일, 환경 변수, 명령줄 인수 등 다양한 외부 구성 소스를 사용할 수 있습니다.

속성 값은 @Value 애노테이션을 사용하여 직접 빈에 주입하거나, Spring의 Environment 추상화를 통해 접근하거나, @ConfigurationProperties를 통해 구조화된 객체에 바인딩할 수 있습니다.

Spring Boot는 값의 합리적인 재정의를 허용하도록 설계된 매우 특정한 PropertySource 순서를 사용합니다. 나중에 정의된 속성 소스가 이전에 정의된 값을 재정의할 수 있습니다. 소스는 다음 순서로 고려됩니다:

1. 기본 속성 (SpringApplication.setDefaultProperties(Map)을 설정하여 지정).
2. @Configuration 클래스의 @PropertySource 애노테이션. 이러한 속성 소스는 애플리케이션 컨텍스트가 새로 고쳐질 때까지 Environment에 추가되지 않으므로, 새로 고침이 시작되기 전에 읽히는 logging.* 및 spring.main.*과 같은 특정 속성을 구성하기에는 너무 늦습니다.
3. 구성 데이터 (예: application.properties 파일).
4. random.*에만 속성이 있는 RandomValuePropertySource.
5. OS 환경 변수.
6. Java 시스템 속성 (System.getProperties()).
7. java:comp/env의 JNDI 속성.
8. ServletContext 초기화 매개변수.
9. ServletConfig 초기화 매개변수.
10. SPRING_APPLICATION_JSON의 속성 (환경 변수 또는 시스템 속성에 내장된 인라인 JSON).
11. 명령줄 인수.
12. 테스트의 properties 속성. @SpringBootTest 및 애플리케이션의 특정 슬라이스를 테스트하기 위한 테스트 애노테이션에서 사용 가능.
13. 테스트의 @DynamicPropertySource 애노테이션.
14. 테스트의 @TestPropertySource 애노테이션.
15. devtools가 활성화된 경우 $HOME/.config/spring-boot 디렉토리의 Devtools 전역 설정 속성.

구성 데이터 파일은 다음 순서로 고려됩니다:

1. jar 내부에 패키징된 애플리케이션 속성 (application.properties 및 YAML 변형).
2. jar 내부에 패키징된 프로필별 애플리케이션 속성 (application-{profile}.properties 및 YAML 변형).
3. 패키징된 jar 외부의 애플리케이션 속성 (application.properties 및 YAML 변형).
4. 패키징된 jar 외부의 프로필별 애플리케이션 속성 (application-{profile}.properties 및 YAML 변형).

애플리케이션 전체에 하나의 형식을 사용하는 것이 좋습니다. 동일한 위치에 .properties와 YAML 형식의 구성 파일이 있는 경우 .properties가 우선합니다. 시스템 속성 대신 환경 변수를 사용하는 경우 대부분의 운영 체제는 점으로 구분된 키 이름을 허용하지 않지만 대신 밑줄을 사용할 수 있습니다 (예: spring.config.name 대신 SPRING_CONFIG_NAME). 환경 변수에서 바인딩에 대한 자세한 내용은 [여기](#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables)를 참조하세요. 애플리케이션이 서블릿 컨테이너나 애플리케이션 서버에서 실행되는 경우, JNDI 속성 (java:comp/env) 또는 서블릿 컨텍스트 초기화 매개변수를 환경 변수나 시스템 속성 대신 사용할 수 있습니다.

구체적인 예를 제공하기 위해, 다음 예제와 같이 name 속성을 사용하는 @Component를 개발한다고 가정해 보겠습니다:

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```

애플리케이션 클래스 경로 (예: jar 내부)에 name에 대한 합리적인 기본 속성 값을 제공하는 application.properties 파일을 가질 수 있습니다. 새로운 환경에서 실행할 때, jar 외부에 제공된 application.properties 파일이 name을 재정의할 수 있습니다. 일회성 테스트를 위해, 특정 명령줄 스위치 (예: `java -jar app.jar --name="Spring"`)로 실행할 수 있습니다.

env 및 configprops 엔드포인트는 속성이 특정 값을 가지는 이유를 파악하는 데 유용할 수 있습니다. 이 두 엔드포인트를 사용하여 예상치 못한 속성 값을 진단할 수 있습니다. 자세한 내용은 [Production ready features](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html) 섹션을 참조하세요.

## 명령줄 속성 접근

기본적으로, SpringApplication은 모든 명령줄 옵션 인수 (즉, `--server.port=9000`과 같은 `--`로 시작하는 인수)를 속성으로 변환하고 이를 Spring Environment에 추가합니다. 앞서 언급했듯이, 명령줄 속성은 항상 파일 기반 속성 소스보다 우선합니다.

명령줄 속성을 Environment에 추가하지 않으려면, `SpringApplication.setAddCommandLineProperties(false)`를 사용하여 비활성화할 수 있습니다.

## JSON 애플리케이션 속성

환경 변수와 시스템 속성은 종종 일부 속성 이름을 사용할 수 없다는 제한이 있습니다. 이를 돕기 위해, Spring Boot는 속성 블록을 단일 JSON 구조로 인코딩할 수 있도록 합니다.

애플리케이션이 시작될 때, 모든 `spring.application.json` 또는 `SPRING_APPLICATION_JSON` 속성이 구문 분석되어 Environment에 추가됩니다.

예를 들어, `SPRING_APPLICATION_JSON` 속성은 UN*X 셸에서 환경 변수로 명령줄에 제공될 수 있습니다:
[UN\*X](https://en.wiktionary.org/wiki/UN*X)

```shell
$ SPRING_APPLICATION_JSON='{"my":{"name":"test"}}' java -jar myapp.jar
```

위의 예에서, Spring Environment에는 `my.name=test`가 설정됩니다.

동일한 JSON은 시스템 속성으로도 제공될 수 있습니다:

```shell
$ java -Dspring.application.json='{"my":{"name":"test"}}' -jar myapp.jar
```

또는 명령줄 인수를 사용하여 JSON을 제공할 수 있습니다:

```shell
$ java -jar myapp.jar --spring.application.json='{"my":{"name":"test"}}'
```

클래식 애플리케이션 서버에 배포하는 경우, `java:comp/env/spring.application.json`이라는 JNDI 변수를 사용할 수도 있습니다.

JSON의 null 값은 결과 속성 소스에 추가되지만, `PropertySourcesPropertyResolver`는 null 속성을 누락된 값으로 처리합니다. 이는 JSON이 null 값으로 하위 순서 속성 소스의 속성을 재정의할 수 없음을 의미합니다.

## 외부 애플리케이션 속성

Spring Boot는 애플리케이션이 시작될 때 다음 위치에서 `application.properties` 및 `application.yaml` 파일을 자동으로 찾고 로드합니다:

- 클래스 경로에서
  - 클래스 경로 루트
  - 클래스 경로 `/config` 패키지
- 현재 디렉토리에서
  - 현재 디렉토리
  - 현재 디렉토리의 `config/` 하위 디렉토리
  - `config/` 하위 디렉토리의 즉시 하위 디렉토리

목록은 우선 순위에 따라 정렬됩니다 (하위 항목의 값이 이전 항목을 재정의). 로드된 파일의 문서는 Spring Environment에 `PropertySource` 인스턴스로 추가됩니다.

`application`을 구성 파일 이름으로 사용하고 싶지 않다면, `spring.config.name` 환경 속성을 지정하여 다른 파일 이름으로 전환할 수 있습니다. 예를 들어, `myproject.properties` 및 `myproject.yaml` 파일을 찾으려면 다음과 같이 애플리케이션을 실행할 수 있습니다:

```shell
$ java -jar myproject.jar --spring.config.name=myproject
```

명시적 위치를 참조하려면 `spring.config.location` 환경 속성을 사용할 수 있습니다. 이 속성은 확인할 하나 이상의 위치를 쉼표로 구분된 목록으로 허용합니다.

다음 예제는 두 개의 별개의 파일을 지정하는 방법을 보여줍니다:

```shell
$ java -jar myproject.jar --spring.config.location=\
    optional:classpath:/default.properties,\
    optional:classpath:/override.properties
```

위치가 선택 사항이고 존재하지 않아도 괜찮다면, `optional:` 접두사를 사용하세요. `spring.config.name`, `spring.config.location`, 및 `spring.config.additional-location`은 로드해야 할 파일을 결정하기 위해 매우 일찍 사용됩니다. 이들은 환경 속성 (일반적으로 OS 환경 변수, 시스템 속성, 또는 명령줄 인수)으로 정의되어야 합니다.

`spring.config.location`에 디렉토리 (파일이 아닌)가 포함된 경우, 이들은 `/`로 끝나야 합니다. 런타임 시, `spring.config.name`에서 생성된 이름이 추가되어 로드됩니다. `spring.config.location`에 지정된 파일은 직접 가져옵니다.

디렉토리 및 파일 위치 값은 프로필별 파일을 확인하기 위해 확장됩니다. 예를 들어, `spring.config.location`이 `classpath:myconfig.properties`인 경우, 적절한 `classpath:myconfig-<profile>.properties` 파일도 로드됩니다.

대부분의 상황에서, 추가하는 각 `spring.config.location` 항목은 단일 파일 또는 디렉토리를 참조합니다. 위치는 정의된 순서대로 처리되며, 나중에 정의된 항목이 이전 항목의 값을 재정의할 수 있습니다.

복잡한 위치 설정이 있고, 프로필별 구성 파일을 사용하는 경우, Spring Boot가 이들이 어떻게 그룹화되어야 하는지 알 수 있도록 추가 힌트를 제공해야 할 수 있습니다. 위치 그룹은 모두 동일한 수준으로 간주되는 위치의 모음입니다. 예를 들어, 모든 클래스 경로 위치를 그룹화한 다음 모든 외부 위치를 그룹화할 수 있습니다. 위치 그룹 내의 항목은 `;`로 구분되어야 합니다. 자세한 내용은 [프로필별 파일](#프로필별-파일) 섹션의 예제를 참조하세요.

`spring.config.location`을 사용하여 구성된 위치는 기본 위치를 대체합니다. 예를 들어, `spring.config.location`이 `optional:classpath:/custom-config/,optional:file:./custom-config/` 값으로 구성된 경우, 고려되는 전체 위치 집합은 다음과 같습니다:

- `optional:classpath:custom-config/`
- `optional:file:./custom-config/`

추가 위치를 추가하는 것을 선호한다면, `spring.config.additional-location`을 사용할 수 있습니다. 추가 위치에서 로드된 속성은 기본 위치의 속성을 재정의할 수 있습니다. 예를 들어, `spring.config.additional-location`이 `optional:classpath:/custom-config/,optional:file:./custom-config/` 값으로 구성된 경우, 고려되는 전체 위치 집합은 다음과 같습니다:

- `optional:classpath:/;optional:classpath:/config/`
- `optional:file:./;optional:file:./config/;optional:file:./config/*/`
- `optional:classpath:custom-config/`
- `optional:file:./custom-config/`

이 검색 순서는 하나의 구성 파일에서 기본값을 지정한 다음 다른 파일에서 선택적으로 해당 값을 재정의할 수 있도록 합니다. 애플리케이션의 기본값을 `application.properties` (또는 `spring.config.name`으로 선택한 다른 기본 이름) 중 하나의 기본 위치에 제공할 수 있습니다. 그런 다음 이러한 기본값은 사용자 지정 위치 중 하나에 있는 다른 파일로 런타임에 재정의될 수 있습니다.

### 선택적 위치

기본적으로, 지정된 구성 데이터 위치가 존재하지 않으면, Spring Boot는 `ConfigDataLocationNotFoundException`을 발생시키고 애플리케이션이 시작되지 않습니다.

위치를 지정하고 싶지만 항상 존재하지 않아도 괜찮다면, `optional:` 접두사를 사용할 수 있습니다. 이 접두사는 `spring.config.location` 및 `spring.config.additional-location` 속성뿐만 아니라 `spring.config.import` 선언에서도 사용할 수 있습니다.

예를 들어, `spring.config.import` 값이 `optional:file:./myconfig.properties`인 경우, `myconfig.properties` 파일이 없어도 애플리케이션이 시작됩니다.

모든 `ConfigDataLocationNotFoundException` 오류를 무시하고 항상 애플리케이션을 계속 시작하려면, `spring.config.on-not-found` 속성을 사용할 수 있습니다. `SpringApplication.setDefaultProperties(…​)` 또는 시스템/환경 변수를 사용하여 값을 `ignore`로 설정하세요.

### 와일드카드 위치

구성 파일 위치에 마지막 경로 세그먼트로 `*` 문자가 포함된 경우, 이는 와일드카드 위치로 간주됩니다. 구성 로드 시 와일드카드가 확장되어 즉시 하위 디렉토리도 확인됩니다. 와일드카드 위치는 여러 구성 속성 소스가 있는 Kubernetes와 같은 환경에서 특히 유용합니다.

예를 들어, Redis 구성과 MySQL 구성이 있는 경우, 이 두 구성 요소를 별도로 유지하면서도 둘 다 `application.properties` 파일에 존재해야 할 수 있습니다. 이는 `/config/redis/application.properties` 및 `/config/mysql/application.properties`와 같은 다른 위치에 두 개의 별도의 `application.properties` 파일이 마운트되는 결과를 초래할 수 있습니다. 이 경우, `config/*/`의 와일드카드 위치를 사용하면 두 파일이 모두 처리됩니다.

기본적으로, Spring Boot는 기본 검색 위치에 `config/*/`을 포함합니다. 이는 jar 외부의 `/config` 디렉토리의 모든 하위 디렉토리가 검색됨을 의미합니다.

와일드카드 위치는 `spring.config.location` 및 `spring.config.additional-location` 속성을 사용하여 직접 사용할 수 있습니다.

와일드카드 위치는 디렉토리 검색 위치의 경우 `*/`로 끝나거나 파일 검색 위치의 경우 `*/<filename>`로 끝나야 합니다. 와일드카드가 있는 위치는 파일 이름의 절대 경로를 기준으로 알파벳순으로 정렬됩니다.

와일드카드 위치는 외부 디렉토리에서만 작동합니다. 클래스 경로 위치에서는 와일드카드를 사용할 수 없습니다.

### 프로필별 파일

애플리케이션 속성 파일뿐만 아니라, Spring Boot는 `application-{profile}` 명명 규칙을 사용하여 프로필별 파일도 로드하려고 시도합니다. 예를 들어, 애플리케이션이 `prod`라는 프로필을 활성화하고 YAML 파일을 사용하는 경우, `application.yaml` 및 `application-prod.yaml`이 고려됩니다.

프로필별 속성은 표준 `application.properties`와 동일한 위치에서 로드되며, 프로필별 파일은 항상 비특정 파일을 재정의합니다. 여러 프로필이 지정된 경우, 마지막으로 지정된 프로필이 우선합니다. 예를 들어, `spring.profiles.active` 속성에 의해 `prod,live` 프로필이 지정된 경우, `application-prod.properties`의 값은 `application-live.properties`의 값에 의해 재정의될 수 있습니다.

마지막으로 지정된 프로필이 우선하는 전략은 위치 그룹 수준에서 적용됩니다. `spring.config.location`이 `classpath:/cfg/,classpath:/ext/`인 경우, `classpath:/cfg/;classpath:/ext/`와 동일한 재정의 규칙을 가지지 않습니다.

예를 들어, 위의 `prod,live` 예제를 계속해서 다음과 같은 파일이 있을 수 있습니다:

- /cfg
  - application-live.properties
- /ext
  - application-live.properties
  - application-prod.properties

`spring.config.location`이 `classpath:/cfg/,classpath:/ext/`인 경우, 모든 /cfg 파일을 /ext 파일보다 먼저 처리합니다:

- /cfg/application-live.properties
- /ext/application-prod.properties
- /ext/application-live.properties

대신 `classpath:/cfg/;classpath:/ext/` (구분 기호로 `;` 사용)인 경우, /cfg 및 /ext를 동일한 수준에서 처리합니다:

- /ext/application-prod.properties
- /cfg/application-live.properties
- /ext/application-live.properties

Environment에는 활성 프로필이 설정되지 않은 경우 사용되는 기본 프로필 집합이 있습니다 (기본적으로 `[default]`). 즉, 명시적으로 프로필이 활성화되지 않은 경우, `application-default`의 속성이 고려됩니다.

속성 파일은 한 번만 로드됩니다. 프로필별 속성 파일을 직접 가져온 경우, 두 번째로 가져오지 않습니다.

### 구성 파일 가져오기

애플리케이션 구성 파일은 `spring.config.import` 속성을 사용하여 추가 구성 데이터를 가져올 수 있습니다. 가져오기는 처리되는 즉시 로드되며, 가져오는 파일의 내용이 가져오기 선언 바로 다음에 처리됩니다. 예를 들어, 클래스 경로의 application.properties 파일에 다음과 같은 내용이 있다고 가정해 보겠습니다:

```properties
spring.application.name=myapp
spring.config.import=optional:file:./dev.properties
```

이는 현재 디렉토리에서 dev.properties 파일을 가져오도록 지시합니다. dev.properties의 값은 가져오기 선언 이후에 처리됩니다. 예를 들어, dev.properties에 다음과 같은 내용이 있다면:

```properties
spring.application.name=mydev-app
```

myapp 이름은 mydev-app으로 재정의됩니다.

여러 위치를 가져오려면 쉼표로 구분된 목록을 지정하세요:

```properties
spring.config.import=optional:file:./dev.properties,optional:classpath:/settings.properties
```

가져오기는 선언된 순서대로 처리됩니다. 위의 예에서는 dev.properties가 먼저 가져와지고 그 다음에 settings.properties가 가져와집니다.

가져온 파일에는 자체 `spring.config.import` 속성이 포함될 수 있으며, 이를 통해 계층적 가져오기가 가능합니다. 가져오기가 순환 참조를 생성하는 경우 오류가 발생합니다.

### 확장자가 없는 파일 가져오기

일부 클라우드 플랫폼에서는 볼륨 마운트된 파일에 확장자를 추가할 수 없습니다. 이러한 확장자가 없는 파일을 가져오려면 Spring Boot에 힌트를 제공하여 파일을 로드하는 방법을 알려줘야 합니다. 이는 대괄호 안에 확장자 힌트를 넣어 수행할 수 있습니다.

예를 들어, yaml로 가져오고 싶은 `/etc/config/myconfig` 파일이 있다고 가정해 보겠습니다. 다음과 같이 application.properties에서 이 파일을 가져올 수 있습니다:

**Properties**

```properties
spring.config.import=file:/etc/config/myconfig[.yaml]
```

**YAML**

```yaml
spring:
  config:
    import: "file:/etc/config/myconfig[.yaml]"
```

### 구성 트리 사용하기

클라우드 플랫폼(예: Kubernetes)에서 애플리케이션을 실행할 때 플랫폼이 제공하는 구성 값을 읽어야 하는 경우가 많습니다. 이러한 목적으로 환경 변수를 사용하는 것이 일반적이지만, 특히 값을 비밀로 유지해야 하는 경우 단점이 있을 수 있습니다.

환경 변수의 대안으로, 많은 클라우드 플랫폼은 이제 구성을 마운트된 데이터 볼륨에 매핑할 수 있도록 합니다. 예를 들어, Kubernetes는 ConfigMap과 Secret을 모두 볼륨 마운트할 수 있습니다.

일반적으로 사용되는 두 가지 볼륨 마운트 패턴이 있습니다:

1. 단일 파일에 전체 속성 집합이 포함됩니다(일반적으로 YAML로 작성됨).
2. 여러 파일이 디렉토리 트리에 작성되며, 파일 이름이 '키'가 되고 내용이 '값'이 됩니다.

첫 번째 경우에는 위에서 설명한 대로 `spring.config.import`를 사용하여 YAML 또는 Properties 파일을 직접 가져올 수 있습니다. 두 번째 경우에는 Spring Boot가 모든 파일을 속성으로 노출해야 함을 알 수 있도록 `configtree:` 접두사를 사용해야 합니다.

예를 들어, Kubernetes가 다음과 같은 볼륨을 마운트했다고 가정해 보겠습니다:

```
etc/
  config/
    myapp/
      username
      password
```

username 파일의 내용은 구성 값이고, password의 내용은 비밀입니다.

이러한 속성을 가져오려면 application.properties 또는 application.yaml 파일에 다음을 추가할 수 있습니다:

**Properties**

```properties
spring.config.import=optional:configtree:/etc/config/
```

**YAML**

```yaml
spring:
  config:
    import: "optional:configtree:/etc/config/"
```

그런 다음 일반적인 방식으로 Environment에서 `myapp.username` 및 `myapp.password` 속성에 접근하거나 주입할 수 있습니다.

> 구성 트리 아래의 폴더와 파일 이름이 속성 이름을 형성합니다. 위의 예에서 속성을 `username` 및 `password`로 접근하려면 `spring.config.import`를 `optional:configtree:/etc/config/myapp`으로 설정할 수 있습니다.

> 점 표기법이 있는 파일 이름도 올바르게 매핑됩니다. 예를 들어, 위의 예에서 `/etc/config`에 `myapp.username`이라는 파일이 있으면 Environment에 `myapp.username` 속성이 생성됩니다.

> 구성 트리 값은 예상되는 내용에 따라 문자열 `String` 및 바이트 배열 `byte[]` 유형 모두에 바인딩될 수 있습니다.

동일한 상위 폴더에서 여러 구성 트리를 가져와야 하는 경우 와일드카드 단축키를 사용할 수 있습니다. `/*`/로 끝나는 `configtree:` 위치는 모든 직계 하위 항목을 구성 트리로 가져옵니다. 와일드카드가 아닌 가져오기와 마찬가지로, 각 구성 트리 아래의 폴더와 파일 이름이 속성 이름을 형성합니다.

예를 들어, 다음과 같은 볼륨이 있다고 가정해 보겠습니다:

```
etc/
  config/
    dbconfig/
      db/
        username
        password
    mqconfig/
      mq/
        username
        password
```

가져오기 위치로 `configtree:/etc/config/*/`를 사용할 수 있습니다:

**Properties**

```properties
spring.config.import=optional:configtree:/etc/config/*/
```

**YAML**

```yaml
spring:
  config:
    import: "optional:configtree:/etc/config/*/"
```

이렇게 하면 `db.username`, `db.password`, `mq.username` 및 `mq.password` 속성이 추가됩니다.

> 와일드카드를 사용하여 로드된 디렉토리는 알파벳순으로 정렬됩니다. 다른 순서가 필요한 경우 각 위치를 별도의 가져오기로 나열해야 합니다.

구성 트리는 Docker 비밀에도 사용할 수 있습니다. Docker 스웜 서비스가 비밀에 접근할 수 있는 권한을 부여받으면 비밀이 컨테이너에 마운트됩니다. 예를 들어, `db.password`라는 비밀이 `/run/secrets/` 위치에 마운트된 경우 다음과 같이 Spring 환경에서 `db.password`를 사용할 수 있습니다:

**Properties**

```properties
spring.config.import=optional:configtree:/run/secrets/
```

**YAML**

```yaml
spring:
  config:
    import: "optional:configtree:/run/secrets/"
```

### 속성 플레이스홀더

application.properties 및 application.yml에 정의된 값은 기존 Environment를 통해 필터링되므로, 이전에 정의된 값을 참조할 수 있습니다. 표준 ${name} 속성 플레이스홀더 구문을 사용할 수 있습니다:

```properties
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

또한 이 기술을 사용하여 기존 Spring Boot 속성에 대한 짧은 플레이스홀더를 만들 수 있습니다. 자세한 내용은 [howto.html](https://docs.spring.io/spring-boot/reference/howto/properties-and-configuration.html#howto.properties-and-configuration.short-command-line-arguments)을 참조하세요.

플레이스홀더 값을 찾을 수 없는 경우, ${my-value} 같은 플레이스홀더는 해결되지 않습니다. 속성이 존재하지 않을 때 기본값을 제공하려면 ${my-value:default}와 같은 구문을 사용하세요:

```properties
app.name=MyApp
app.description=${app.name} is a Spring Boot application running on ${server.port:8080}
```

이 접근 방식은 기본값이 하드코딩되어 있거나 다른 속성에서 계산되는 경우에 특히 유용합니다. 기본적으로 모든 플레이스홀더는 반드시 해결되어야 하며, 플레이스홀더를 해결할 수 없는 경우 예외가 발생합니다. 이 동작은 다음과 같이 spring.config.on-unresolved-placeholder를 설정하여 변경할 수 있습니다:

| 모드 | 설명 |
| --- | --- |
| error | 예외를 발생시킵니다. |
| ignore | 플레이스홀더를 무시합니다. |
| warn | 경고를 기록하고 계속 진행합니다. |

### 다중 문서 파일 작업하기

Spring Boot를 사용하면 단일 물리적 파일을 여러 논리적 문서로 분할할 수 있으며, 각 문서는 독립적으로 추가됩니다. 문서는 위에서 아래로 순서대로 처리됩니다. 나중에 오는 문서는 이전 문서에 정의된 속성을 재정의할 수 있습니다.

application.yaml 파일의 경우 표준 YAML 다중 문서 구문이 사용됩니다. 연속된 세 개의 하이픈은 한 문서의 끝과 다음 문서의 시작을 나타냅니다.

예를 들어, 다음 파일에는 두 개의 논리적 문서가 있습니다:

```yaml
spring:
  application:
    name: "MyApp"
---
spring:
  application:
    name: "MyCloudApp"
  config:
    activate:
      on-cloud-platform: "kubernetes"
```

application.properties 파일의 경우 특수한 `#---` 또는 `!---` 주석을 사용하여 문서 분할을 표시합니다:

```properties
spring.application.name=MyApp
#---
spring.application.name=MyCloudApp
spring.config.activate.on-cloud-platform=kubernetes
```

> 속성 파일 구분자 앞에는 공백이 없어야 하며 정확히 세 개의 하이픈 문자가 있어야 합니다. 구분자 바로 앞과 뒤의 줄은 동일한 주석 접두사가 아니어야 합니다.

> 다중 문서 속성 파일은 종종 `spring.config.activate.on-profile`과 같은 활성화 속성과 함께 사용됩니다. 자세한 내용은 다음 섹션을 참조하세요.

> 다중 문서 속성 파일은 `@PropertySource` 또는 `@TestPropertySource` 애노테이션을 사용하여 로드할 수 없습니다.

### 활성화 속성

특정 조건이 충족될 때만 주어진 속성 집합을 활성화하는 것이 유용할 수 있습니다. 예를 들어, 특정 프로필이 활성화된 경우에만 관련이 있는 속성이 있을 수 있습니다.

`spring.config.activate.*`를 사용하여 속성 문서를 조건부로 활성화할 수 있습니다.

다음과 같은 활성화 속성을 사용할 수 있습니다:

**표 1. 활성화 속성**

| 속성 | 설명 |
| --- | --- |
| on-profile | 문서가 활성화되기 위해 일치해야 하는 프로필 표현식입니다. |
| on-cloud-platform | 문서가 활성화되기 위해 감지되어야 하는 CloudPlatform입니다. |

예를 들어, 다음은 두 번째 문서가 Kubernetes에서 실행 중이고 "prod" 또는 "staging" 프로필이 활성화된 경우에만 활성화됨을 지정합니다:

**Properties**

```properties
myprop=always-set
#---
spring.config.activate.on-cloud-platform=kubernetes
spring.config.activate.on-profile=prod | staging
myotherprop=sometimes-set
```

**YAML**

```yaml
myprop: "always-set"
---
spring:
  config:
    activate:
      on-cloud-platform: "kubernetes"
      on-profile: "prod | staging"
myotherprop: "sometimes-set"
```

## YAML 작업하기

YAML은 JSON의 상위 집합으로, 계층적 구성 데이터를 지정하기에 편리한 형식입니다. SpringApplication 클래스는 클래스패스에 SnakeYAML 라이브러리가 있을 때마다 속성의 대안으로 YAML을 자동으로 지원합니다.

스타터를 사용하는 경우, SnakeYAML은 spring-boot-starter에 의해 자동으로 제공됩니다.

### YAML을 속성으로 매핑하기

YAML 문서는 계층적 형식에서 Spring Environment에서 사용할 수 있는 평면 구조로 변환되어야 합니다. 예를 들어, 다음 YAML 문서를 고려해 보세요:

```yaml
environments:
  dev:
    url: "https://dev.example.com"
    name: "Developer Setup"
  prod:
    url: "https://another.example.com"
    name: "My Cool App"
```

Environment에서 이러한 속성에 접근하기 위해, 다음과 같이 평면화됩니다:

```properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

마찬가지로, YAML 목록도 평면화되어야 합니다. 이들은 [인덱스] 역참조자가 있는 속성 키로 표현됩니다. 예를 들어, 다음 YAML을 고려해 보세요:

```yaml
my:
  servers:
  - "dev.example.com"
  - "another.example.com"
```

위의 예는 다음과 같은 속성으로 변환됩니다:

```properties
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

[인덱스] 표기법을 사용하는 속성은 Spring Boot의 Binder 클래스를 사용하여 Java List 또는 Set 객체에 바인딩될 수 있습니다. 자세한 내용은 아래의 타입 안전 구성 속성 섹션을 참조하세요.

> YAML 파일은 @PropertySource 또는 @TestPropertySource 애노테이션을 사용하여 로드할 수 없습니다. 따라서 그런 방식으로 값을 로드해야 하는 경우, 속성 파일을 사용해야 합니다.

### YAML 직접 로드하기

Spring Framework는 YAML 문서를 로드하는 데 사용할 수 있는 두 가지 편리한 클래스를 제공합니다. YamlPropertiesFactoryBean은 YAML을 Properties로 로드하고, YamlMapFactoryBean은 YAML을 Map으로 로드합니다.

YAML을 Spring PropertySource로 로드하려면 YamlPropertySourceLoader 클래스를 사용할 수도 있습니다.

## 임의 값 구성하기

RandomValuePropertySource는 임의 값(예: 비밀 또는 테스트 케이스)을 주입하는 데 유용합니다. 다음 예제와 같이 정수, long, uuid 또는 문자열을 생성할 수 있습니다:

**Properties**

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number-less-than-ten=${random.int(10)}
my.number-in-range=${random.int[1024,65536]}
```

**YAML**

```yaml
my:
  secret: "${random.value}"
  number: "${random.int}"
  bignumber: "${random.long}"
  uuid: "${random.uuid}"
  number-less-than-ten: "${random.int(10)}"
  number-in-range: "${random.int[1024,65536]}"
```

random.int* 구문은 OPEN value (,max) CLOSE 형식이며, 여기서 OPEN,CLOSE는 임의의 문자이고 value,max는 정수입니다. max가 제공되면, value는 최소값이고 max는 최대값(배타적)입니다.

## 시스템 환경 속성 구성하기

Spring Boot는 환경 속성에 접두사를 설정하는 것을 지원합니다. 이는 시스템 환경이 서로 다른 구성 요구 사항을 가진 여러 Spring Boot 애플리케이션에서 공유되는 경우에 유용합니다. 시스템 환경 속성의 접두사는 SpringApplication에 직접 설정할 수 있습니다.

예를 들어, 접두사를 input으로 설정하면, remote.timeout과 같은 속성은 시스템 환경에서 input.remote.timeout으로도 해석됩니다.

## 암호화된 속성 사용

Spring Boot는 암호화된 속성 값을 지원하지 않지만, Spring Boot 애플리케이션에 이 기능을 추가하는 데 필요한 후크 포인트를 제공합니다. EnvironmentPostProcessor 인터페이스를 사용하면 애플리케이션이 시작되기 전에 Environment를 조작할 수 있습니다. 자세한 내용은 [howto.html](https://docs.spring.io/spring-boot/reference/howto/application.html#howto.application.customize-the-environment-or-application-context)을 참조하세요.

암호화된 속성 값이 필요한 경우, [Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/)를 살펴보세요. 이는 외부 애플리케이션 구성을 위한 서버 및 클라이언트 측 지원을 제공합니다. 서버는 암호화 및 암호 해독을 위한 안전한 환경을 제공하며, 클라이언트는 Spring Environment에 통합됩니다.

### 타입 안전 구성 속성 작업

@Value("${property}") 애노테이션을 사용하여 구성 속성을 주입하는 것은 때로는 번거로울 수 있으며, 특히 여러 속성을 사용하거나 데이터가 계층적으로 구조화된 경우에 그렇습니다. Spring Boot는 구조화된 환경 속성을 관리하고 유효성을 검사하는 대체 방법을 제공합니다.

### JavaBean 속성 바인딩

예를 들어, 다음과 같은 속성을 정의했다고 가정해 보겠습니다:

```properties
my.service.remote-address=192.168.1.1
my.service.security.username=admin
my.service.security.password=secret
```

이러한 속성을 사용하는 `@ConfigurationProperties` 빈을 만들 수 있습니다:

```java
import java.net.InetAddress;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("my.service")
public class MyProperties {

    private InetAddress remoteAddress;

    private final Security security = new Security();

    // getter와 setter...

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public void setRemoteAddress(InetAddress remoteAddress) {
        this.remoteAddress = remoteAddress;
    }

    public Security getSecurity() {
        return this.security;
    }

    public static class Security {

        private String username;

        private String password;

        // getter와 setter...

        public String getUsername() {
            return this.username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public String getPassword() {
            return this.password;
        }

        public void setPassword(String password) {
            this.password = password;
        }

    }

}
```
## JavaBean 속성 바인딩

다음과 같은 속성들을 정의할 수 있습니다:

- `my.service.enabled`: 기본값이 false인 속성
- `my.service.remote-address`: String에서 변환될 수 있는 타입의 속성
- `my.service.security.username`: 속성 이름에 의해 결정되는 중첩된 "security" 객체. 특히 여기서는 타입이 전혀 사용되지 않으며 SecurityProperties일 수도 있습니다.
- `my.service.security.password`
- `my.service.security.roles`: 기본값이 USER인 String 컬렉션

속성 이름에 `import`와 같은 예약어를 사용하려면, 속성의 필드에 `@Name` 애노테이션을 사용하세요.

Spring Boot에서 사용 가능한 `@ConfigurationProperties` 클래스에 매핑되는 속성들은 속성 파일, YAML 파일, 환경 변수 및 기타 메커니즘을 통해 구성되며 공개 API이지만, 클래스 자체의 접근자(getter/setter)는 직접 사용하기 위한 것이 아닙니다.

이러한 구성은 기본 빈 생성자에 의존하며, 바인딩이 Spring MVC와 마찬가지로 표준 Java Beans 속성 설명자를 통해 이루어지기 때문에 getter와 setter가 일반적으로 필수입니다. 다음과 같은 경우에는 setter를 생략할 수 있습니다:

- Map은 초기화되어 있는 한 getter가 필요하지만 반드시 setter가 필요하지는 않습니다. 바인더에 의해 변경될 수 있기 때문입니다.
- 컬렉션과 배열은 인덱스(일반적으로 YAML 사용 시)를 통해 접근하거나 단일 쉼표로 구분된 값(properties)을 사용하여 접근할 수 있습니다. 후자의 경우 setter가 필수입니다. 이러한 타입에는 항상 setter를 추가하는 것이 좋습니다. 컬렉션을 초기화하는 경우 변경 불가능하지 않도록 해야 합니다(앞의 예제와 같이).
- 중첩된 POJO 속성이 초기화된 경우(앞의 예제에서 Security 필드와 같이) setter가 필요하지 않습니다. 바인더가 기본 생성자를 사용하여 인스턴스를 즉시 생성하도록 하려면 setter가 필요합니다.

일부 개발자는 Project Lombok을 사용하여 getter와 setter를 자동으로 추가합니다. Lombok이 해당 타입에 대해 특정 생성자를 생성하지 않도록 해야 합니다. 컨테이너가 객체를 인스턴스화하는 데 자동으로 사용되기 때문입니다.

마지막으로, 표준 Java Bean 속성만 고려되며 정적 속성에 대한 바인딩은 지원되지 않습니다.

### 생성자 바인딩

이전 섹션의 예제는 다음과 같이 불변 방식으로 다시 작성할 수 있습니다:

```java
import java.net.InetAddress;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.bind.DefaultValue;

@ConfigurationProperties("my.service")
public class MyProperties {

    // 필드...

    private final boolean enabled;

    private final InetAddress remoteAddress;

    private final Security security;


    public MyProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    // getter...

    public boolean isEnabled() {
        return this.enabled;
    }

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public Security getSecurity() {
        return this.security;
    }

    public static class Security {

        // 필드...

        private final String username;

        private final String password;

        private final List<String> roles;


        public Security(String username, String password, @DefaultValue("USER") List<String> roles) {
            this.username = username;
            this.password = password;
            this.roles = roles;
        }

        // getter...

        public String getUsername() {
            return this.username;
        }

        public String getPassword() {
            return this.password;
        }

        public List<String> getRoles() {
            return this.roles;
        }

    }
}
```

이 설정에서는 매개변수가 있는 단일 생성자가 있으면 생성자 바인딩을 사용해야 함을 의미합니다. 즉, 바인더는 바인딩하려는 매개변수가 있는 생성자를 찾습니다. 클래스에 여러 생성자가 있는 경우 `@ConstructorBinding` 애노테이션을 사용하여 생성자 바인딩에 사용할 생성자를 지정할 수 있습니다. 매개변수가 있는 단일 생성자가 있는 클래스에 대해 생성자 바인딩을 사용하지 않으려면 생성자에 `@Autowired`로 주석을 달거나 private으로 만들어야 합니다. 생성자 바인딩은 레코드와 함께 사용할 수 있습니다. 레코드에 여러 생성자가 없는 한 `@ConstructorBinding`을 사용할 필요가 없습니다.

생성자 바인딩 클래스의 중첩 멤버(위 예제의 Security와 같은)도 생성자를 통해 바인딩됩니다.

기본값은 생성자 매개변수 및 레코드 컴포넌트에 `@DefaultValue`를 사용하여 지정할 수 있습니다. 변환 서비스는 누락된 속성의 대상 타입에 애노테이션의 String 값을 강제 변환하는 데 적용됩니다.

이전 예제를 참조하면, Security에 바인딩된 속성이 없는 경우 MyProperties 인스턴스는 security에 대해 null 값을 포함합니다. 속성이 바인딩되지 않은 경우에도 Security의 비 null 인스턴스를 포함하도록 하려면(Kotlin을 사용할 때는 기본값이 없으므로 Security의 username 및 password 매개변수를 nullable로 선언해야 함) 빈 `@DefaultValue` 애노테이션을 사용하세요:

```java
public MyProperties(boolean enabled, InetAddress remoteAddress, @DefaultValue Security security) {
    this.enabled = enabled;
    this.remoteAddress = remoteAddress;
    this.security = security;
}
```

생성자 바인딩을 사용하려면 클래스가 `@EnableConfigurationProperties` 또는 구성 속성 스캐닝을 사용하여 활성화되어야 합니다. 일반 Spring 메커니즘으로 생성된 빈(예: `@Component` 빈, `@Bean` 메서드를 사용하여 생성된 빈 또는 `@Import`를 사용하여 로드된 빈)에서는 생성자 바인딩을 사용할 수 없습니다.

생성자 바인딩을 사용하려면 클래스가 `-parameters`로 컴파일되어야 합니다. Spring Boot의 Gradle 플러그인을 사용하거나 Maven과 spring-boot-starter-parent를 사용하는 경우 자동으로 이루어집니다.

`@ConfigurationProperties`와 함께 Optional을 사용하는 것은 주로 반환 타입으로 사용하기 위한 것이므로 권장되지 않습니다. 따라서 구성 속성 주입에는 적합하지 않습니다. 다른 타입의 속성과의 일관성을 위해 Optional 속성을 선언하고 값이 없는 경우 빈 Optional이 아닌 null이 바인딩됩니다.

속성 이름에 `import`와 같은 예약어를 사용하려면 생성자 매개변수에 `@Name` 애노테이션을 사용하세요.

### @ConfigurationProperties 주석이 달린 타입 활성화

Spring Boot는 `@ConfigurationProperties` 타입을 바인딩하고 빈으로 등록하는 인프라를 제공합니다. 클래스별로 구성 속성을 활성화하거나 컴포넌트 스캐닝과 유사한 방식으로 작동하는 구성 속성 스캐닝을 활성화할 수 있습니다.

때로는 `@ConfigurationProperties`로 주석이 달린 클래스가 스캐닝에 적합하지 않을 수 있습니다. 예를 들어, 자체 자동 구성을 개발하거나 조건부로 활성화하려는 경우입니다. 이러한 경우 `@EnableConfigurationProperties` 애노테이션을 사용하여 처리할 타입 목록을 지정하세요. 이는 다음 예제와 같이 모든 `@Configuration` 클래스에서 수행할 수 있습니다:

```java
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(SomeProperties.class)
public class MyConfiguration {

}
```

```java
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("some.properties")
public class SomeProperties {

}
```

구성 속성 스캐닝을 사용하려면 애플리케이션에 `@ConfigurationPropertiesScan` 애노테이션을 추가하세요. 일반적으로 `@SpringBootApplication`으로 주석이 달린 메인 애플리케이션 클래스에 추가되지만 모든 `@Configuration` 클래스에 추가할 수 있습니다. 기본적으로 스캐닝은 애노테이션을 선언하는 클래스의 패키지에서 발생합니다. 스캔할 특정 패키지를 정의하려면 다음 예제와 같이 할 수 있습니다:

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.ConfigurationPropertiesScan;

@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "com.example.another" })
public class MyApplication {

}
```

`@ConfigurationProperties` 빈이 구성 속성 스캐닝 또는 `@EnableConfigurationProperties`를 통해 등록되면 빈은 규칙적인 이름을 갖습니다: `<prefix>-<fqn>`, 여기서 `<prefix>`는 `@ConfigurationProperties` 애노테이션에 지정된 환경 키 접두사이고 `<fqn>`은 빈의 정규화된 이름입니다. 애노테이션이 접두사를 제공하지 않는 경우 빈의 정규화된 이름만 사용됩니다.

com.example.app 패키지에 있다고 가정하면, 위의 SomeProperties 예제의 빈 이름은 some.properties-com.example.app.SomeProperties입니다.

`@ConfigurationProperties`는 환경만 다루고, 특히 컨텍스트의 다른 빈을 주입하지 않는 것이 좋습니다. 특수한 경우에는 setter 주입을 사용하거나 프레임워크에서 제공하는 *Aware 인터페이스(예: Environment에 접근해야 하는 경우 EnvironmentAware) 중 하나를 사용할 수 있습니다. 여전히 생성자를 사용하여 다른 빈을 주입하려면 구성 속성 빈에 `@Component`로 주석을 달고 JavaBean 기반 속성 바인딩을 사용해야 합니다.

### @ConfigurationProperties 주석이 달린 타입 사용하기

이 스타일의 구성은 다음 예제와 같이 SpringApplication 외부 YAML 구성과 특히 잘 작동합니다:

```yaml
my:
  service:
    remote-address: 192.168.1.1
    security:
      username: "admin"
      roles:
      - "USER"
      - "ADMIN"
```

`@ConfigurationProperties` 빈으로 작업하려면 다음 예제와 같이 다른 빈과 동일한 방식으로 주입할 수 있습니다:

```java
import org.springframework.stereotype.Service;

@Service
public class MyService {

    private final MyProperties properties;

    public MyService(MyProperties properties) {
        this.properties = properties;
    }

    public void openConnection() {
        Server server = new Server(this.properties.getRemoteAddress());
        server.start();
        // ...
    }

    // ...

}
```

`@ConfigurationProperties`를 사용하면 IDE가 자체 키에 대한 자동 완성을 제공하는 데 사용할 수 있는 메타데이터 파일을 생성할 수도 있습니다. 자세한 내용은 부록을 참조하세요.

### 서드파티 구성

클래스에 `@ConfigurationProperties`로 주석을 다는 것 외에도 공개 `@Bean` 메서드에도 사용할 수 있습니다. 이는 특히 제어할 수 없는 서드파티 컴포넌트에 속성을 바인딩하려는 경우에 유용할 수 있습니다.

Environment 속성에서 빈을 구성하려면 다음 예제와 같이 빈 등록에 `@ConfigurationProperties`를 추가하세요:

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class ThirdPartyConfiguration {

    @Bean
    @ConfigurationProperties(prefix = "another")
    public AnotherComponent anotherComponent() {
        return new AnotherComponent();
    }

}
```

another 접두사로 정의된 모든 JavaBean 속성은 앞의 SomeProperties 예제와 유사한 방식으로 해당 AnotherComponent 빈에 매핑됩니다.

### 느슨한 바인딩

Spring Boot는 Environment 속성을 `@ConfigurationProperties` 빈에 바인딩하기 위해 일부 느슨한 규칙을 사용하므로 Environment 속성 이름과 빈 속성 이름이 정확히 일치할 필요가 없습니다. 이것이 유용한 일반적인 예로는 대시로 구분된 환경 속성(예: context-path는 contextPath에 바인딩됨) 및 대문자 환경 속성(예: PORT는 port에 바인딩됨)이 있습니다.

예를 들어, 다음과 같은 `@ConfigurationProperties` 클래스를 고려해 보세요:

```java
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "my.main-project.person")
public class MyPersonProperties {

    private String firstName;

    public String getFirstName() {
        return this.firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

}
```

위의 코드에서는 다음과 같은 속성 이름을 모두 사용할 수 있습니다:

**표 2. 느슨한 바인딩**

| 속성 | 참고 |
| --- | --- |
| my.main-project.person.first-name | .properties 및 YAML 파일에서 사용하는 것이 권장되는 케밥 케이스 |
| my.main-project.person.firstName | 표준 카멜 케이스 구문 |
| my.main-project.person.first_name | .properties 및 YAML 파일에서 사용할 수 있는 대체 형식인 언더스코어 표기법 |
| MY_MAINPROJECT_PERSON_FIRSTNAME | 시스템 환경 변수를 사용할 때 권장되는 대문자 형식 |

애노테이션의 접두사 값은 케밥 케이스(소문자 및 -로 구분, 예: my.main-project.person)여야 합니다.

**표 3. 속성 소스별 느슨한 바인딩 규칙**

| 속성 소스 | 단순 | 목록 |
| --- | --- | --- |
| 속성 파일 | 카멜 케이스, 케밥 케이스 또는 언더스코어 표기법 | [ ]를 사용하는 표준 목록 구문 또는 쉼표로 구분된 값 |
| YAML 파일 | 카멜 케이스, 케밥 케이스 또는 언더스코어 표기법 | 표준 YAML 목록 구문 또는 쉼표로 구분된 값 |
| 환경 변수 | 구분 기호로 언더스코어를 사용하는 대문자 형식(환경 변수에서 바인딩 참조) | 언더스코어로 둘러싸인 숫자(환경 변수에서 바인딩 참조) |
| 시스템 속성 | 카멜 케이스, 케밥 케이스 또는 언더스코어 표기법 | [ ]를 사용하는 표준 목록 구문 또는 쉼표로 구분된 값 |

가능한 경우 속성을 my.person.first-name=Rod와 같은 소문자 케밥 형식으로 저장하는 것이 좋습니다.

#### 맵 바인딩

Map 속성에 바인딩할 때 원래 키 값을 보존하기 위해 특수 대괄호 표기법을 사용해야 할 수 있습니다. 키가 []로 둘러싸여 있지 않으면 영숫자, - 또는 .이 아닌 모든 문자가 제거됩니다.

예를 들어, 다음 속성을 Map<String,String>에 바인딩하는 것을 고려해 보세요:

**Properties**

```properties
my.map[/key1]=value1
my.map[/key2]=value2
my.map./key3=value3
```

**YAML**

```yaml
my:
  map:
    "[/key1]": "value1"
    "[/key2]": "value2"
    "/key3": "value3"
```

YAML 파일의 경우 키가 제대로 구문 분석되도록 대괄호를 따옴표로 둘러싸야 합니다.

위의 속성은 /key1, /key2 및 key3를 맵의 키로 하는 Map에 바인딩됩니다. 슬래시는 대괄호로 둘러싸여 있지 않았기 때문에 key3에서 제거되었습니다.

스칼라 값에 바인딩할 때 키에 .이 있는 경우 []로 둘러싸지 않아도 됩니다. 스칼라 값에는 열거형과 Object를 제외한 java.lang 패키지의 모든 타입이 포함됩니다. a.b=c를 Map<String, String>에 바인딩하면 키의 .이 보존되고 {"a.b"="c"} 항목이 있는 Map이 반환됩니다. 다른 타입의 경우 키에 .이 포함되어 있으면 대괄호 표기법을 사용해야 합니다. 예를 들어, a.b=c를 Map<String, Object>에 바인딩하면 {"a"={"b"="c"}} 항목이 있는 Map이 반환되지만 [a.b]=c는 {"a.b"="c"} 항목이 있는 Map을 반환합니다.

#### 환경 변수에서 바인딩하기

대부분의 운영 체제는 환경 변수 이름에 사용할 수 있는 이름에 대해 엄격한 규칙을 적용합니다. 예를 들어, Linux 셸 변수는 문자(a부터 z 또는 A부터 Z), 숫자(0부터 9) 또는 밑줄 문자(_)만 포함할 수 있습니다. 관례적으로 Unix 셸 변수는 이름이 대문자로 표기됩니다.

Spring Boot의 느슨한 바인딩 규칙은 가능한 한 이러한 이름 제한과 호환되도록 설계되었습니다.

정식 형식의 속성 이름을 환경 변수 이름으로 변환하려면 다음 규칙을 따를 수 있습니다:

1. 점(.)을 밑줄(_)로 바꿉니다.
2. 모든 대시(-)를 제거합니다.
3. 대문자로 변환합니다.

예를 들어, 구성 속성 spring.main.log-startup-info는 SPRING_MAIN_LOGSTARTUPINFO라는 환경 변수가 됩니다.

환경 변수는 객체 목록에 바인딩할 때도 사용할 수 있습니다. List에 바인딩하려면 변수 이름에서 요소 번호를 밑줄로 둘러싸야 합니다.

예를 들어, 구성 속성 my.service[0].other는 MY_SERVICE_0_OTHER라는 환경 변수를 사용합니다.

환경 변수에서 바인딩하기 위한 지원은 systemEnvironment 속성 소스와 이름이 -systemEnvironment로 끝나는 모든 추가 속성 소스에 적용됩니다.

#### 환경 변수에서 맵 바인딩하기

Spring Boot가 환경 변수를 속성 클래스에 바인딩할 때, 바인딩하기 전에 환경 변수 이름을 소문자로 변환합니다. 대부분의 경우 이 세부 사항은 중요하지 않지만, Map 속성에 바인딩할 때는 예외입니다.

Map의 키는 항상 소문자이며, 다음 예제에서 볼 수 있습니다:

```java
import java.util.HashMap;
import java.util.Map;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "my.props")
public class MyMapsProperties {

    private final Map<String, String> values = new HashMap<>();

    public Map<String, String> getValues() {
        return this.values;
    }

}
```

MY_PROPS_VALUES_KEY=value를 설정할 때, values 맵에는 {"key"="value"} 항목이 포함됩니다.

환경 변수 이름만 소문자로 변환되고 값은 변환되지 않습니다. MY_PROPS_VALUES_KEY=VALUE를 설정할 때, values 맵에는 {"key"="VALUE"} 항목이 포함됩니다.

#### 캐싱

느슨한 바인딩은 성능을 향상시키기 위해 캐시를 사용합니다. 기본적으로 이 캐싱은 불변 속성 소스에만 적용됩니다. 이 동작을 사용자 정의하려면, 예를 들어 가변 속성 소스에 대한 캐싱을 활성화하려면 ConfigurationPropertyCaching을 사용하세요.

### 복잡한 타입 병합하기

목록이 둘 이상의 위치에서 구성된 경우, 재정의는 전체 목록을 대체하는 방식으로 작동합니다.

예를 들어, 기본적으로 name과 description 속성이 null인 MyPojo 객체가 있다고 가정해 보겠습니다. 다음 예제는 MyProperties에서 MyPojo 객체 목록을 노출합니다:

```java
import java.util.ArrayList;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("my")
public class MyProperties {

    private final List<MyPojo> list = new ArrayList<>();

    public List<MyPojo> getList() {
        return this.list;
    }

}
```

다음 구성을 고려해 보세요:

**Properties**

```properties
my.list[0].name=my name
my.list[0].description=my description
#---
spring.config.activate.on-profile=dev
my.list[0].name=my another name
```

**YAML**

```yaml
my:
  list:
    - name: "my name"
      description: "my description"
---
spring:
  config:
    activate:
      on-profile: "dev"
my:
  list:
    - name: "my another name"
```

dev 프로필이 활성화되지 않은 경우, MyProperties.list에는 이전에 정의된 대로 하나의 MyPojo 항목이 포함됩니다. 그러나 dev 프로필이 활성화된 경우, 목록에는 여전히 하나의 항목만 포함됩니다(이름이 "my another name"이고 설명이 null인). 이 구성은 목록에 두 번째 MyPojo 인스턴스를 추가하지 않으며 항목을 병합하지도 않습니다.

List가 여러 프로필에 지정된 경우, 우선 순위가 가장 높은 것(그리고 그것만)이 사용됩니다. 다음 예제를 고려해 보세요:

**Properties**

```properties
my.list[0].name=my name
my.list[0].description=my description
my.list[1].name=another name
my.list[1].description=another description
#---
spring.config.activate.on-profile=dev
my.list[0].name=my another name
```

**YAML**

```yaml
my:
  list:
    - name: "my name"
      description: "my description"
    - name: "another name"
      description: "another description"
---
spring:
  config:
    activate:
      on-profile: "dev"
my:
  list:
    - name: "my another name"
```

위의 예에서 dev 프로필이 활성화된 경우, MyProperties.list에는 하나의 MyPojo 항목이 포함됩니다(이름이 "my another name"이고 설명이 null인). YAML의 경우, 쉼표로 구분된 목록과 YAML 목록 모두 목록의 내용을 완전히 재정의하는 데 사용할 수 있습니다.

Map 속성의 경우, 여러 소스에서 가져온 속성 값으로 바인딩할 수 있습니다. 그러나 여러 소스에서 동일한 속성의 경우, 우선 순위가 가장 높은 것이 사용됩니다. 다음 예제는 MyProperties에서 Map<String, MyPojo>를 노출합니다:

```java
import java.util.LinkedHashMap;
import java.util.Map;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("my")
public class MyProperties {

    private final Map<String, MyPojo> map = new LinkedHashMap<>();

    public Map<String, MyPojo> getMap() {
        return this.map;
    }

}
```

다음 구성을 고려해 보세요:

**Properties**

```properties
my.map.key1.name=my name 1
my.map.key1.description=my description 1
#---
spring.config.activate.on-profile=dev
my.map.key1.name=dev name 1
my.map.key2.name=dev name 2
my.map.key2.description=dev description 2
```

**YAML**

```yaml
my:
  map:
    key1:
      name: "my name 1"
      description: "my description 1"
---
spring:
  config:
    activate:
      on-profile: "dev"
my:
  map:
    key1:
      name: "dev name 1"
    key2:
      name: "dev name 2"
      description: "dev description 2"
```

dev 프로필이 활성화되지 않은 경우, MyProperties.map에는 key1 키가 있는 하나의 항목이 포함됩니다(이름이 "my name 1"이고 설명이 "my description 1"인). 그러나 dev 프로필이 활성화된 경우, map에는 key1(이름이 "dev name 1"이고 설명이 "my description 1"인)과 key2(이름이 "dev name 2"이고 설명이 "dev description 2"인) 키가 있는 두 개의 항목이 포함됩니다.

앞서 언급한 병합 규칙은 모든 속성 소스의 속성에 적용되며, 파일에만 국한되지 않습니다.

### 속성 변환

Spring Boot는 @ConfigurationProperties 빈에 바인딩할 때 외부 애플리케이션 속성을 올바른 타입으로 강제 변환하려고 시도합니다. 사용자 정의 타입 변환이 필요한 경우, ConversionService 빈(conversionService라는 이름의 빈)이나 사용자 정의 속성 편집기(CustomEditorConfigurer 빈을 통해) 또는 사용자 정의 변환기(@ConfigurationPropertiesBinding으로 주석이 달린 빈 정의)를 제공할 수 있습니다.

이 빈은 애플리케이션 수명 주기 초기에 요청되므로, ConversionService가 사용하는 종속성을 제한해야 합니다. 일반적으로 필요한 종속성은 생성 시점에 완전히 초기화되지 않을 수 있습니다. 구성 키 강제 변환에 필요하지 않고 @ConfigurationPropertiesBinding으로 한정된 사용자 정의 변환기에만 의존하는 경우 사용자 정의 ConversionService의 이름을 변경할 수 있습니다.

#### 기간 변환하기

Spring Boot는 기간 표현을 위한 전용 지원을 제공합니다. Duration 속성을 노출하는 경우, application properties에서 다음 형식을 사용할 수 있습니다:

- 일반 long 표현(기본 단위로 밀리초를 사용하지만 @DurationUnit이 지정된 경우 제외)
- Duration에서 사용하는 표준 ISO-8601 형식
- 값과 단위가 결합된 더 읽기 쉬운 형식(10s는 10초를 의미함)

다음 예제를 고려해 보세요:

```java
import java.time.Duration;
import java.time.temporal.ChronoUnit;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.convert.DurationUnit;

@ConfigurationProperties("my")
public class MyProperties {

    @DurationUnit(ChronoUnit.SECONDS)
    private Duration sessionTimeout = Duration.ofSeconds(30);

    private Duration readTimeout = Duration.ofMillis(1000);

    // getters / setters...

    public Duration getSessionTimeout() {
        return this.sessionTimeout;
    }

    public void setSessionTimeout(Duration sessionTimeout) {
        this.sessionTimeout = sessionTimeout;
    }

    public Duration getReadTimeout() {
        return this.readTimeout;
    }

    public void setReadTimeout(Duration readTimeout) {
        this.readTimeout = readTimeout;
    }

}
```

30초의 세션 타임아웃을 지정하려면 30, PT30S, 30s는 모두 동일합니다. 500ms의 읽기 타임아웃은 다음 형식 중 하나로 지정할 수 있습니다: 500, PT0.5S, 500ms.

지원되는 모든 단위를 사용할 수도 있습니다. 이들은 다음과 같습니다:

- ns: 나노초
- us: 마이크로초
- ms: 밀리초
- s: 초
- m: 분
- h: 시간
- d: 일

기본 단위는 밀리초이며, 위 예제에서 보여진 것처럼 @DurationUnit을 사용하여 재정의할 수 있습니다.

생성자 바인딩을 선호하는 경우, 다음 예제와 같이 동일한 속성을 노출할 수 있습니다:

```java
import java.time.Duration;
import java.time.temporal.ChronoUnit;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.bind.DefaultValue;
import org.springframework.boot.convert.DurationUnit;

@ConfigurationProperties("my")
public class MyProperties {

    // fields...
    private final Duration sessionTimeout;

    private final Duration readTimeout;

    public MyProperties(@DurationUnit(ChronoUnit.SECONDS) @DefaultValue("30s") Duration sessionTimeout,
            @DefaultValue("1000ms") Duration readTimeout) {
        this.sessionTimeout = sessionTimeout;
        this.readTimeout = readTimeout;
    }

    // getters...

    public Duration getSessionTimeout() {
        return this.sessionTimeout;
    }

    public Duration getReadTimeout() {
        return this.readTimeout;
    }

}
```

Long 속성을 업그레이드하는 경우, 밀리초가 아니라면 단위(@DurationUnit 사용)를 정의해야 합니다. 이렇게 하면 훨씬 더 풍부한 형식을 지원하면서 투명한 업그레이드 경로를 제공합니다.

#### 기간 변환하기

기간 외에도 Spring Boot는 Period 타입과도 작동할 수 있습니다. application properties에서 다음 형식을 사용할 수 있습니다:

- 일반 int 표현(기본 단위로 일을 사용하지만 @PeriodUnit이 지정된 경우 제외)
- Period에서 사용하는 표준 ISO-8601 형식
- 값과 단위 쌍이 결합된 더 간단한 형식(1y3d는 1년과 3일을 의미함)

다음 단위는 간단한 형식에서 지원됩니다:

- y: 년
- m: 월
- w: 주
- d: 일

Period 타입은 실제로 주 수를 저장하지 않으며, "7일"을 의미하는 단축어입니다.

#### 데이터 크기 변환하기

Spring Framework에는 바이트 단위로 크기를 표현하는 DataSize 값 타입이 있습니다. DataSize 속성을 노출하는 경우, application properties에서 다음 형식을 사용할 수 있습니다:

- 일반 long 표현(기본 단위로 바이트를 사용하지만 @DataSizeUnit이 지정된 경우 제외)
- 값과 단위가 결합된 더 읽기 쉬운 형식(10MB는 10메가바이트를 의미함)

다음 예제를 고려해 보세요:

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.convert.DataSizeUnit;
import org.springframework.util.unit.DataSize;
import org.springframework.util.unit.DataUnit;

@ConfigurationProperties("my")
public class MyProperties {

    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize bufferSize = DataSize.ofMegabytes(2);

    private DataSize sizeThreshold = DataSize.ofBytes(512);

    // getters/setters...

    public DataSize getBufferSize() {
        return this.bufferSize;
    }

    public void setBufferSize(DataSize bufferSize) {
        this.bufferSize = bufferSize;
    }

    public DataSize getSizeThreshold() {
        return this.sizeThreshold;
    }

    public void setSizeThreshold(DataSize sizeThreshold) {
        this.sizeThreshold = sizeThreshold;
    }

}
```

10메가바이트의 버퍼 크기를 지정하려면 10과 10MB는 동일합니다. 256바이트의 크기 임계값은 256 또는 256B로 지정할 수 있습니다.

지원되는 모든 단위를 사용할 수도 있습니다. 이들은 다음과 같습니다:

- B: 바이트
- KB: 킬로바이트
- MB: 메가바이트
- GB: 기가바이트
- TB: 테라바이트

기본 단위는 바이트이며, 위 예제에서 보여진 것처럼 @DataSizeUnit을 사용하여 재정의할 수 있습니다.

생성자 바인딩을 선호하는 경우, 다음 예제와 같이 동일한 속성을 노출할 수 있습니다:

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.bind.DefaultValue;
import org.springframework.boot.convert.DataSizeUnit;
import org.springframework.util.unit.DataSize;
import org.springframework.util.unit.DataUnit;

@ConfigurationProperties("my")
public class MyProperties {

    // fields...
    private final DataSize bufferSize;

    private final DataSize sizeThreshold;

    public MyProperties(@DataSizeUnit(DataUnit.MEGABYTES) @DefaultValue("2MB") DataSize bufferSize,
            @DefaultValue("512B") DataSize sizeThreshold) {
        this.bufferSize = bufferSize;
        this.sizeThreshold = sizeThreshold;
    }

    // getters...

    public DataSize getBufferSize() {
        return this.bufferSize;
    }

    public DataSize getSizeThreshold() {
        return this.sizeThreshold;
    }

}
```

Long 속성을 업그레이드하는 경우, 바이트가 아니라면 단위(@DataSizeUnit 사용)를 정의해야 합니다. 이렇게 하면 훨씬 더 풍부한 형식을 지원하면서 투명한 업그레이드 경로를 제공합니다.

#### Base64 데이터 변환하기

Spring Boot는 Base64로 인코딩된 이진 데이터를 해석하는 것을 지원합니다. Resource 속성을 노출하는 경우, 다음 예제와 같이 base64: 접두사가 있는 값으로 base64 인코딩된 텍스트를 제공할 수 있습니다:

**Properties**

```properties
my.property=base64:SGVsbG8gV29ybGQ=
```

**YAML**

```yaml
my:
  property: "base64:SGVsbG8gV29ybGQ="
```

Resource 속성은 리소스 경로를 제공하는 데도 사용할 수 있어 더 다양하게 활용할 수 있습니다.

### @ConfigurationProperties 검증

Spring Boot는 @ConfigurationProperties 클래스가 Spring의 @Validated 애노테이션으로 주석 처리될 때마다 검증을 시도합니다. JSR-303 jakarta.validation 제약 조건 애노테이션을 구성 클래스에 직접 사용할 수 있습니다. 이렇게 하려면 클래스패스에 호환되는 JSR-303 구현이 있는지 확인한 다음 다음 예제와 같이 필드에 제약 조건 애노테이션을 추가하세요:

```java
import java.net.InetAddress;

import jakarta.validation.constraints.NotNull;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.validation.annotation.Validated;

@ConfigurationProperties("my.service")
@Validated
public class MyProperties {

    @NotNull
    private InetAddress remoteAddress;

    // getters/setters...

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public void setRemoteAddress(InetAddress remoteAddress) {
        this.remoteAddress = remoteAddress;
    }

}
```

구성 속성을 생성하는 @Bean 메서드에 @Validated로 주석을 달아 검증을 트리거할 수도 있습니다.

중첩된 속성에 대한 검증을 계단식으로 적용하려면 연관된 필드에 @Valid로 주석을 달아야 합니다. 다음 예제는 앞서 언급한 MyProperties 예제를 기반으로 합니다:

```java
import java.net.InetAddress;

import jakarta.validation.Valid;
import jakarta.validation.constraints.NotEmpty;
import jakarta.validation.constraints.NotNull;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.validation.annotation.Validated;

@ConfigurationProperties("my.service")
@Validated
public class MyProperties {

    @NotNull
    private InetAddress remoteAddress;

    @Valid
    private final Security security = new Security();

    // getters/setters...

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public void setRemoteAddress(InetAddress remoteAddress) {
        this.remoteAddress = remoteAddress;
    }

    public Security getSecurity() {
        return this.security;
    }

    public static class Security {

        @NotEmpty
        private String username;

        // getters/setters...

        public String getUsername() {
            return this.username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

    }

}
```

configurationPropertiesValidator라는 빈 정의를 생성하여 사용자 정의 Spring Validator를 추가할 수도 있습니다. @Bean 메서드는 static으로 선언되어야 합니다. 구성 속성 검증기는 애플리케이션 수명 주기 초기에 생성되며, @Bean 메서드를 static으로 선언하면 @Configuration 클래스를 인스턴스화하지 않고도 빈을 생성할 수 있습니다. 이렇게 하면 조기 인스턴스화로 인해 발생할 수 있는 문제를 방지할 수 있습니다.

spring-boot-actuator 모듈에는 모든 @ConfigurationProperties 빈을 노출하는 엔드포인트가 포함되어 있습니다. 웹 브라우저를 /actuator/configprops로 지정하거나 동등한 JMX 엔드포인트를 사용하세요. 자세한 내용은 프로덕션 준비 기능 섹션을 참조하세요.

### @ConfigurationProperties vs. @Value

@Value 애노테이션은 핵심 컨테이너 기능이며, 타입 안전 구성 속성과 동일한 기능을 제공하지 않습니다. 다음 표는 @ConfigurationProperties와 @Value에서 지원하는 기능을 요약합니다:

| 기능 | @ConfigurationProperties | @Value |
| --- | --- | --- |
| 느슨한 바인딩 | 예 | 제한적(아래 참고 사항 참조) |
| 메타데이터 지원 | 예 | 아니오 |
| SpEL 평가 | 아니오 | 예 |

@Value를 사용하려는 경우, 속성 이름을 정식 형식(소문자만 사용하는 케밥 케이스)으로 참조하는 것이 좋습니다. 이렇게 하면 Spring Boot가 @ConfigurationProperties에서 느슨한 바인딩을 수행할 때와 동일한 로직을 사용할 수 있습니다.

예를 들어, @Value("${demo.item-price}")는 application.properties 파일에서 demo.item-price와 demo.itemPrice 형식을 모두 가져오며, 시스템 환경에서 DEMO_ITEMPRICE도 가져옵니다. 대신 @Value("${demo.itemPrice}")를 사용하면 demo.item-price와 DEMO_ITEMPRICE는 고려되지 않습니다.

자체 구성 요소에 대한 구성 키 집합을 정의하는 경우, @ConfigurationProperties로 주석이 달린 POJO에 그룹화하는 것이 좋습니다. 이렇게 하면 자체 빈에 주입할 수 있는 구조화된 타입 안전 객체를 제공합니다.

애플리케이션 속성 파일의 SpEL 표현식은 이러한 파일을 구문 분석하고 환경을 채울 때 처리되지 않습니다. 그러나 @Value에 SpEL 표현식을 작성하는 것은 가능합니다. 애플리케이션 속성 파일의 속성 값이 SpEL 표현식인 경우, @Value를 통해 소비될 때 평가됩니다.
