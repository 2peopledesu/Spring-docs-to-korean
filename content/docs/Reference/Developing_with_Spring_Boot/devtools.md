---
title: Developer Tools
date: 2025-02-21
weight: 9
---

## 개발자 도구

Spring Boot는 애플리케이션 개발 경험을 더욱 즐겁게 만들어주는 추가 도구들을 포함하고 있습니다. `spring-boot-devtools` 모듈은 어떤 프로젝트에든 포함시켜 추가적인 개발 시간 기능을 제공할 수 있습니다. devtools 지원을 포함하려면 다음과 같이 Maven과 Gradle용 빌드에 모듈 의존성을 추가하세요:

**Maven**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

**Gradle**
```gradle
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

> **🔥 주의**
> 
> Devtools는 특히 멀티 모듈 프로젝트에서 클래스로딩 문제를 일으킬 수 있습니다. [클래스로딩 문제 진단하기](#클래스로딩-문제-진단하기)에서 이를 진단하고 해결하는 방법을 설명합니다.

> **⚠️ 주의**
> 
> 개발자 도구는 완전히 패키징된 애플리케이션을 실행할 때 자동으로 비활성화됩니다. `java -jar`로 애플리케이션을 실행하거나 특별한 클래스로더에서 시작하는 경우 "프로덕션 애플리케이션"으로 간주됩니다. `spring.devtools.restart.enabled` 시스템 속성을 사용하여 이 동작을 제어할 수 있습니다. 애플리케이션을 시작하는 데 사용되는 클래스로더와 관계없이 devtools를 활성화하려면 `-Dspring.devtools.restart.enabled=true` 시스템 속성을 설정하세요. 이는 devtools 실행이 보안 위험이 될 수 있는 프로덕션 환경에서는 수행해서는 안 됩니다. devtools를 비활성화하려면 의존성을 제외하거나 `-Dspring.devtools.restart.enabled=false` 시스템 속성을 설정하세요.

> **💡 참고**
> 
> Maven에서 의존성을 optional로 표시하거나 Gradle에서 `developmentOnly` 설정을 사용하면(위에 표시된 대로) devtools가 프로젝트를 사용하는 다른 모듈에 전이적으로 적용되는 것을 방지합니다.

> **💡 참고**
> 
> 리패키지된 아카이브는 기본적으로 devtools를 포함하지 않습니다. [특정 원격 devtools 기능](#원격-애플리케이션)을 사용하려면 이를 포함해야 합니다. Maven 플러그인을 사용할 때는 `excludeDevtools` 속성을 `false`로 설정하세요. Gradle 플러그인을 사용할 때는 [태스크의 클래스패스를 구성하여 developmentOnly 설정을 포함](../../../build-tool-plugin/gradle-plugin/packaging.md#packaging-executable.configuring.including-development-only-dependencies)하세요.

## 클래스로딩 문제 진단하기

[재시작 vs 리로드](#재시작-vs-리로드) 섹션에서 설명한 대로, 재시작 기능은 두 개의 클래스로더를 사용하여 구현됩니다. 대부분의 애플리케이션에서 이 접근 방식은 잘 작동합니다. 하지만 특히 멀티 모듈 프로젝트에서는 때때로 클래스로딩 문제가 발생할 수 있습니다.

클래스로딩 문제가 실제로 devtools와 두 개의 클래스로더로 인해 발생하는지 진단하려면, [재시작 기능을 비활성화](#재시작-제외하기
)해보세요. 이렇게 하면 문제가 해결된다면, [재시작 클래스로더를 커스터마이즈](#사용자-정의-재시작-클래스로더)하여 프로젝트 전체를 포함하도록 설정하세요.

## 프로퍼티 기본값

Spring Boot가 지원하는 몇몇 라이브러리는 성능을 향상시키기 위해 캐시를 사용합니다. 예를 들어, [템플릿 엔진](../../web.md#web.servlet.spring-mvc.template-engines)은 템플릿 파일을 반복해서 파싱하지 않도록 컴파일된 템플릿을 캐시합니다. 또한, Spring MVC는 정적 리소스를 제공할 때 응답에 HTTP 캐시 헤더를 추가할 수 있습니다.

캐싱은 프로덕션에서는 매우 유용하지만 개발 중에는 오히려 방해가 될 수 있어서, 변경한 내용을 즉시 확인하지 못하게 합니다. 이러한 이유로 spring-boot-devtools는 기본적으로 캐싱 옵션을 비활성화합니다.

캐시 옵션은 일반적으로 `application.properties` 파일의 설정을 통해 구성됩니다. 예를 들어, Thymeleaf는 `spring.thymeleaf.cache` 속성을 제공합니다. 이러한 속성을 수동으로 설정할 필요 없이 `spring-boot-devtools` 모듈이 자동으로 개발 시간에 적합한 구성을 적용합니다.

다음 표에는 적용되는 모든 속성이 나열되어 있습니다:

| 이름 | 기본값 |
|------|--------|
| server.error.include-binding-errors | always |
| server.error.include-message | always |
| server.error.include-stacktrace | always |
| server.servlet.jsp.init-parameters.development | true |
| server.servlet.session.persistent | true |
| spring.docker.compose.readiness.wait | only-if-started |
| spring.freemarker.cache | false |
| spring.graphql.graphiql.enabled | true |
| spring.groovy.template.cache | false |
| spring.h2.console.enabled | true |
| spring.mustache.servlet.cache | false |
| spring.mvc.log-resolved-exception | true |
| spring.reactor.netty.shutdown-quiet-period | 0s |
| spring.template.provider.cache | false |
| spring.thymeleaf.cache | false |
| spring.web.resources.cache.period | 0 |
| spring.web.resources.chain.cache | false |

> **💡 참고**
> 
> 프로퍼티 기본값을 적용하지 않으려면 `application.properties`에서 `spring.devtools.add-properties`를 `false`로 설정할 수 있습니다.

Spring MVC와 Spring WebFlux 애플리케이션을 개발하는 동안 웹 요청에 대한 더 많은 정보가 필요하기 때문에, 개발자 도구는 `web` 로깅 그룹에 대해 `DEBUG` 로깅을 활성화할 것을 제안합니다. 이를 통해 들어오는 요청, 어떤 핸들러가 처리하는지, 응답 결과 및 기타 세부 정보를 확인할 수 있습니다. 모든 요청 세부 정보(잠재적으로 민감한 정보 포함)를 로깅하려면 `spring.mvc.log-request-details` 또는 `spring.codec.log-request-details` 설정 프로퍼티를 활성화할 수 있습니다.

## 자동 재시작

`spring-boot-devtools`를 사용하는 애플리케이션은 클래스패스의 파일이 변경될 때마다 자동으로 재시작됩니다. IDE에서 작업할 때 매우 유용한 기능으로, 코드 변경에 대한 빠른 피드백을 제공합니다. 기본적으로 디렉토리를 가리키는 클래스패스의 모든 항목이 변경 사항을 모니터링합니다. 정적 리소스와 뷰 템플릿과 같은 특정 리소스는 [애플리케이션을 재시작할 필요가 없다](#재시작-제외하기)는 부분에 유의해야 합니다.

> ### 재시작 트리거하기
>
> DevTools는 클래스패스 리소스를 모니터링하므로, 재시작을 트리거하는 유일한 방법은 클래스패스를 업데이트하는 것입니다. IDE나 빌드 플러그인을 사용하든, 수정된 파일은 재시작을 트리거하기 위해 다시 컴파일되어야 합니다. 클래스패스를 업데이트하는 방법은 사용하는 도구에 따라 다릅니다:
>
> - Eclipse에서는 수정된 파일을 저장하면 클래스패스가 업데이트되고 재시작이 트리거됩니다.
> - IntelliJ IDEA에서는 프로젝트 빌드(`Build -> Build Project`)가 동일한 효과를 가집니다.
> - 빌드 플러그인을 사용하는 경우, Maven은 `mvn compile`, Gradle은 `gradle build`를 실행하면 재시작이 트리거됩니다.

> **💡 참고**
> 
> Maven이나 Gradle을 사용하여 빌드 플러그인으로 재시작하는 경우 forking을 활성화된 상태로 두어야 합니다. forking을 비활성화하면 devtools가 사용하는 격리된 애플리케이션 클래스로더가 생성되지 않아 재시작이 제대로 작동하지 않습니다.

> **💡 참고**
> 
> 자동 재시작은 LiveReload와 함께 사용할 때 매우 잘 작동합니다. 자세한 내용은 [LiveReload](#livereload) 섹션을 참조하세요. JRebel을 사용하는 경우, 동적 클래스 리로딩을 위해 자동 재시작이 비활성화됩니다. 다른 devtools 기능(LiveReload 및 프로퍼티 오버라이드 등)은 계속 사용할 수 있습니다.

> **💡 참고**
> 
> DevTools는 재시작 중에 애플리케이션 컨텍스트를 닫기 위해 종료 훅을 사용합니다. 종료 훅을 비활성화한 경우(`SpringApplication.setRegisterShutdownHook(false)`) 제대로 작동하지 않습니다.

> **💡 참고**
> 
> DevTools는 ApplicationContext가 사용하는 [ResourceLoader](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/core/io/ResourceLoader.html)를 커스터마이즈해야 합니다. 애플리케이션이 이미 ResourceLoader를 제공하는 경우 래핑됩니다. ApplicationContext의 `getResource` 메서드를 직접 오버라이드하는 것은 지원되지 않습니다.

> **💡 참고**
> 
> AspectJ weaving을 사용할 때는 자동 재시작이 지원되지 않습니다.

>### 재시작 vs 리로드
>
>Spring Boot가 제공하는 재시작 기술은 두 개의 클래스로더를 사용합니다. 변경되지 않는 클래스(예: 서드파티 jar)는 *base* 클래스로더에 로드됩니다. 개발 중인 클래스는 *restart* 클래스로더에 로드됩니다. 애플리케이션이 재시작되면 *restart* 클래스로더는 폐기되고 새로운 클래스로더가 생성됩니다. 이러한 접근 방식은 base 클래스로더가 이미 사용 가능한 클래스를 다시 로드할 필요가 없기 때문에 일반적인 "cold starts"보다 애플리케이션 재시작이 훨씬 빠르다는 것을 의미합니다.
>
>애플리케이션의 재시작이 충분히 빠르지 않거나 클래스로더 문제가 발생하는 경우, JRebel과 같은 리로딩 기술을 사용할 수 있습니다. 이러한 접근 방식은 로드된 클래스를 다시 작성하여 새로운 바이트코드로 업데이트합니다.

### Logging Changes in Condition Evaluation

기본적으로 애플리케이션이 재시작될 때마다 조건 평가 델타를 보여주는 보고서가 로깅됩니다. 이 보고서는 빈을 추가하거나 제거하고 설정 프로퍼티를 변경하는 등의 변경사항에 따른 애플리케이션의 자동 구성 변경사항을 보여줍니다.

보고서 로깅을 비활성화하려면 다음 프로퍼티를 설정하세요:

```properties
spring.devtools.restart.log-condition-evaluation-delta=false
```

### 리소스 제외하기

특정 리소스는 변경되어도 반드시 재시작을 트리거할 필요가 없습니다. 예를 들어, Thymeleaf 템플릿은 제자리에서 편집할 수 있습니다. 기본적으로 `/META-INF/maven`, `/META-INF/resources`, `/resources`, `/static`, `/public`, `/templates`의 리소스를 변경해도 재시작을 트리거하지 않지만 LiveReload는 트리거됩니다. 이러한 제외 항목을 커스터마이즈하려면 `spring.devtools.restart.exclude` 프로퍼티를 사용할 수 있습니다. 예를 들어, `/static`과 `/public`만 제외하려면 다음과 같이 설정하세요:

```properties
spring.devtools.restart.exclude=static/**,public/**
```

> **💡 참고**
> 
> 기본 제외 항목을 유지하면서 추가 제외 항목을 설정하려면 대신 `spring.devtools.restart.additional-exclude` 프로퍼티를 사용하세요.

### 추가 경로 감시하기

클래스패스에 없는 파일을 변경할 때도 애플리케이션을 재시작하거나 리로드하고 싶을 수 있습니다. 이를 위해 `spring.devtools.restart.additional-paths` 프로퍼티를 사용하여 변경 사항을 감시할 추가 경로를 구성할 수 있습니다. [앞서 설명한](#리소스-제외하기) `spring.devtools.restart.exclude` 프로퍼티를 사용하여 추가 경로 아래의 변경 사항이 전체 재시작을 트리거할지 또는 LiveReload를 트리거할지 제어할 수 있습니다.

### 재시작 비활성화하기

재시작 기능을 사용하지 않으려면 `spring.devtools.restart.enabled` 프로퍼티를 사용하여 비활성화할 수 있습니다. 대부분의 경우 이 프로퍼티를 `application.properties`에 설정할 수 있습니다(이렇게 하면 재시작 클래스로더는 초기화되지만 파일 변경을 감시하지는 않습니다).

재시작 지원을 완전히 비활성화해야 하는 경우(예: 특정 라이브러리와 작동하지 않는 경우), 다음 예제와 같이 `SpringApplication.run(…​)`을 호출하기 전에 `spring.devtools.restart.enabled` [System](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/System.html) 프로퍼티를 `false`로 설정해야 합니다:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        System.setProperty("spring.devtools.restart.enabled", "false");
        SpringApplication.run(MyApplication.class, args);
    }

}
```
### 트리거 파일 사용하기

IDE가 변경된 파일을 지속적으로 컴파일하는 경우, 특정 시점에만 재시작을 트리거하고 싶을 수 있습니다. 이를 위해 "트리거 파일"을 사용할 수 있습니다. 트리거 파일은 재시작 검사를 실제로 트리거하고 싶을 때 수정해야 하는 특별한 파일입니다.

> **💡 참고**
> 
> 파일을 업데이트하면 검사가 트리거되지만, Devtools가 실제로 수행할 작업이 있다고 감지한 경우에만 재시작이 발생합니다.

트리거 파일을 사용하려면 `spring.devtools.restart.trigger-file` 프로퍼티를 트리거 파일의 이름(경로 제외)으로 설정하세요. 트리거 파일은 클래스패스 어딘가에 있어야 합니다.

예를 들어, 프로젝트가 다음과 같은 구조를 가지고 있다면:

```
src
+- main
   +- resources
      +- .reloadtrigger
```

`trigger-file` 프로퍼티는 다음과 같이 설정됩니다:

```properties
spring.devtools.restart.trigger-file=.reloadtrigger
```

이제 재시작은 `src/main/resources/.reloadtrigger`가 업데이트될 때만 발생합니다.

> **💡 참고**
> 
> 모든 프로젝트가 동일한 방식으로 동작하도록 `spring.devtools.restart.trigger-file`을 [전역 설정](#전역-설정)으로 지정하는 것이 좋습니다.

일부 IDE에는 트리거 파일을 수동으로 업데이트할 필요가 없는 기능이 있습니다. Spring Tools for Eclipse와 IntelliJ IDEA(Ultimate Edition) 모두 이러한 지원을 제공합니다. Spring Tools에서는 콘솔 뷰의 "reload" 버튼을 사용할 수 있습니다(트리거 파일의 이름이 `.reloadtrigger`인 경우). IntelliJ IDEA의 경우 [해당 문서](https://www.jetbrains.com/help/idea/spring-boot.html#application-update-policies)의 지침을 따르면 됩니다.

### 재시작 클래스로더 커스터마이즈하기

재시작 vs 리로드 섹션에서 설명한 대로, 재시작 기능은 두 개의 클래스로더를 사용하여 구현됩니다. 이로 인해 문제가 발생하는 경우, `spring.devtools.restart.enabled` 시스템 프로퍼티를 사용하여 문제를 진단할 수 있으며, 재시작을 비활성화했을 때 앱이 정상 작동한다면 어떤 클래스로더로 무엇을 로드할지 커스터마이즈해야 할 수 있습니다.

기본적으로 IDE에서 열려 있는 모든 프로젝트는 "restart" 클래스로더로 로드되고, 일반 `.jar` 파일은 "base" 클래스로더로 로드됩니다. `mvn spring-boot:run` 또는 `gradle bootRun`을 사용할 때도 마찬가지입니다: `@SpringBootApplication`이 포함된 프로젝트는 "restart" 클래스로더로 로드되고, 나머지는 "base" 클래스로더로 로드됩니다. 앱을 시작할 때 클래스패스가 콘솔에 출력되므로 문제가 있는 항목을 식별하는 데 도움이 될 수 있습니다. 리플렉션으로 사용되는 클래스, 특히 어노테이션은 해당 클래스를 사용하는 애플리케이션 클래스보다 먼저 부모(고정) 클래스로더에 로드될 수 있으며, 이로 인해 Spring이 애플리케이션에서 이를 감지하지 못할 수 있습니다.

`META-INF/spring-devtools.properties` 파일을 생성하여 프로젝트의 일부를 다른 클래스로더로 로드하도록 Spring Boot에 지시할 수 있습니다. `spring-devtools.properties` 파일은 `restart.exclude`와 `restart.include` 접두사가 붙은 프로퍼티를 포함할 수 있습니다. `include` 요소는 "restart" 클래스로더로 끌어올려야 하는 항목이고, exclude 요소는 "base" 클래스로더로 내려야 하는 항목입니다. 프로퍼티의 값은 시작 시 JVM에 전달되는 클래스패스에 적용되는 정규식 패턴입니다. 다음은 일부 로컬 클래스 파일을 제외하고 일부 추가 라이브러리를 재시작 클래스 로더에 포함시키는 예시입니다:

```properties
restart:
  exclude:
    companycommonlibs: "/mycorp-common-[\\w\\d-\\.]/(build|bin|out|target)/"
  include:
    projectcommon: "/mycorp-myproj-[\\w\\d-\\.]+\\.jar"
```

> **💡 참고**
> 
> 모든 프로퍼티 키는 고유해야 합니다. 프로퍼티가 `restart.include.` 또는 `restart.exclude.`로 시작하는 한 해당 속성은 고려됩니다.

> **💡 참고**
> 
> 클래스패스의 모든 `META-INF/spring-devtools.properties`가 로드됩니다. 프로젝트 내부나 프로젝트가 사용하는 라이브러리에 파일을 패키징할 수 있습니다. 시스템 프로퍼티는 사용할 수 없으며, 프로퍼티 파일만 사용할 수 있습니다.

### 알려진 제한사항

재시작 기능은 표준 ObjectInputStream을 사용하여 역직렬화되는 객체와는 잘 작동하지 않습니다. 데이터를 역직렬화해야 하는 경우 Spring의 ConfigurableObjectInputStream을 `Thread.currentThread().getContextClassLoader()`와 함께 사용해야 할 수 있습니다.

안타깝게도 여러 서드파티 라이브러리들이 컨텍스트 클래스로더를 고려하지 않고 역직렬화를 수행합니다. 이러한 문제를 발견하면 원 저자에게 수정을 요청해야 합니다.

## LiveReload

`spring-boot-devtools` 모듈에는 리소스가 변경될 때 브라우저 새로고침을 트리거할 수 있는 내장 LiveReload 서버가 포함되어 있습니다. LiveReload 브라우저 확장 프로그램은 Chrome, Firefox, Safari에서 무료로 사용할 수 있습니다. 선택한 브라우저의 마켓플레이스나 스토어에서 'LiveReload'를 검색하여 이러한 확장 프로그램을 찾을 수 있습니다.

애플리케이션 실행 시 LiveReload 서버를 시작하지 않으려면 `spring.devtools.livereload.enabled` 프로퍼티를 `false`로 설정할 수 있습니다.

> **💡 참고**
> 
> LiveReload 서버는 한 번에 하나만 실행할 수 있습니다. 애플리케이션을 시작하기 전에 다른 LiveReload 서버가 실행 중이지 않은지 확인하세요. IDE에서 여러 애플리케이션을 시작하는 경우 첫 번째 애플리케이션만 LiveReload 지원을 받습니다.

> **💡 참고**
> 
> 파일이 변경될 때 LiveReload를 트리거하려면 [자동 재시작](#자동-재시작)이 활성화되어 있어야 합니다.

## 전역 설정

다음 파일 중 하나를 `$HOME/.config/spring-boot` 디렉토리에 추가하여 전역 devtools 설정을 구성할 수 있습니다:

1. `spring-boot-devtools.properties`
2. `spring-boot-devtools.yaml`
3. `spring-boot-devtools.yml`

이러한 파일에 추가된 프로퍼티는 devtools를 사용하는 모든 Spring Boot 애플리케이션에 적용됩니다. 예를 들어, 재시작이 항상 트리거 파일을 사용하도록 구성하려면 `spring-boot-devtools` 파일에 다음 프로퍼티를 추가하면 됩니다:

```properties
spring.devtools.restart.trigger-file=.reloadtrigger
```

기본적으로 `$HOME`은 사용자의 홈 디렉토리입니다. 이 위치를 커스터마이즈하려면 `SPRING_DEVTOOLS_HOME` 환경 변수나 `spring.devtools.home` 시스템 프로퍼티를 설정하세요.

> **💡 참고**
> 
> devtools 설정 파일이 $HOME/.config/spring-boot에서 발견되지 않으면, $HOME 디렉토리의 루트에서 .spring-boot-devtools.properties 파일의 존재 여부를 검색합니다. 이를 통해 $HOME/.config/spring-boot 위치를 지원하지 않는 이전 버전의 Spring Boot를 사용하는 애플리케이션과 devtools 전역 설정을 공유할 수 있습니다.

> **💡 참고**
> 
> devtools properties/yaml 파일에서는 프로파일이 지원되지 않습니다. `.spring-boot-devtools.properties`에서 활성화된 프로파일은 profile-specific configuration files 로딩에 영향을 미치지 않습니다. `spring-boot-devtools-<profile>.properties` 형식의 프로파일별 파일명과 YAML과 Properties 파일 모두에서 `spring.config.activate.on-profile` 문서는 지원되지 않습니다.

### File System Watcher 구성하기

[FileSystemWatcher](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/devtools/filewatch/FileSystemWatcher.html)는 일정한 시간 간격으로 클래스 변경을 폴링하고, 더 이상의 변경이 없는지 확인하기 위해 미리 정의된 대기 시간을 기다리는 방식으로 작동합니다. Spring Boot는 IDE가 파일을 컴파일하고 Spring Boot가 읽을 수 있는 위치로 복사하는 것에 전적으로 의존하기 때문에, devtools가 애플리케이션을 재시작할 때 특정 변경사항이 반영되지 않는 경우가 있을 수 있습니다. 이러한 문제가 지속적으로 발생한다면, 개발 환경에 맞는 값으로 `spring.devtools.restart.poll-interval`과 `spring.devtools.restart.quiet-period` 매개변수를 증가시켜 보세요:

```properties
spring.devtools.restart.poll-interval=2s
spring.devtools.restart.quiet-period=1s
```

이제 모니터링되는 클래스패스 디렉토리는 2초마다 변경사항을 폴링하고, 추가적인 클래스 변경이 없는지 확인하기 위해 1초의 대기 시간을 유지합니다.

## 원격 애플리케이션

Spring Boot 개발자 도구는 로컬 개발에만 국한되지 않습니다. 원격으로 애플리케이션을 실행할 때도 여러 기능을 사용할 수 있습니다. 원격 지원은 보안 위험이 될 수 있으므로 opt-in 방식입니다. 신뢰할 수 있는 네트워크에서 실행하거나 SSL로 보안이 설정된 경우에만 활성화해야 합니다. 이러한 옵션을 사용할 수 없는 경우 DevTools의 원격 지원을 사용해서는 안 됩니다. 프로덕션 배포에서는 절대 지원을 활성화하지 마세요.

활성화하려면 다음 목록과 같이 리패키지된 아카이브에 `devtools`가 포함되어 있는지 확인해야 합니다:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```

그런 다음 `spring.devtools.remote.secret` 프로퍼티를 설정해야 합니다. 중요한 비밀번호나 시크릿과 마찬가지로, 이 값은 추측하거나 무차별 대입 공격으로 알아낼 수 없도록 고유하고 강력해야 합니다.

원격 devtools 지원은 두 부분으로 제공됩니다: 연결을 수락하는 서버 측 엔드포인트와 IDE에서 실행하는 클라이언트 애플리케이션입니다. `spring.devtools.remote.secret` 프로퍼티가 설정되면 서버 컴포넌트가 자동으로 활성화됩니다. 클라이언트 컴포넌트는 수동으로 실행해야 합니다.

> **💡 참고**
> 
> Spring WebFlux 애플리케이션에서는 원격 devtools가 지원되지 않습니다.

### 원격 클라이언트 애플리케이션 실행하기

원격 클라이언트 애플리케이션은 IDE 내에서 실행되도록 설계되었습니다. 연결하려는 원격 프로젝트와 동일한 클래스패스로 [RemoteSpringApplication](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/devtools/RemoteSpringApplication.html)을 실행해야 합니다. 애플리케이션의 단일 필수 인수는 연결할 원격 URL입니다.

예를 들어, Eclipse나 Spring Tools를 사용하고 있고 Cloud Foundry에 배포한 `my-app`이라는 프로젝트가 있다면 다음과 같이 하면 됩니다:

* `Run` 메뉴에서 `Run Configurations…​`를 선택합니다.
* 새로운 `Java Application` "실행 구성"을 생성합니다.
* `my-app` 프로젝트를 찾습니다.
* [RemoteSpringApplication](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/devtools/RemoteSpringApplication.html)을 메인 클래스로 사용합니다.
* `Program arguments`에 `https://myapp.cfapps.io`를 추가합니다(또는 원격 URL이 무엇이든).

실행 중인 원격 클라이언트는 다음과 같이 보일 수 있습니다:

```
  .   ____          _                                              __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
 =========|_|==============|___/===================================/_/_/_/
 :: Spring Boot Remote ::  (v3.4.3)

2025-02-20T14:15:59.695Z  INFO 126237 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication v3.4.3 using Java 17.0.14 with PID 126237 (/Users/myuser/.m2/repository/org/springframework/boot/spring-boot-devtools/3.4.3/spring-boot-devtools-3.4.3.jar started by myuser in /opt/apps/)
2025-02-20T14:15:59.707Z  INFO 126237 --- [           main] o.s.b.devtools.RemoteSpringApplication   : No active profile set, falling back to 1 default profile: "default"
2025-02-20T14:16:00.152Z  INFO 126237 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2025-02-20T14:16:00.201Z  INFO 126237 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 2.044 seconds (process running for 3.027)
```

> **💡 참고**
> 
> 원격 클라이언트는 실제 애플리케이션과 동일한 클래스패스를 사용하므로 애플리케이션 프로퍼티를 직접 읽을 수 있습니다. 이것이 spring.devtools.remote.secret 프로퍼티를 읽어서 인증을 위해 서버로 전달하는 방식입니다.

> **💡 참고**
> 
> 트래픽이 암호화되고 비밀번호가 가로채지지 않도록 항상 https://를 연결 프로토콜로 사용하는 것이 좋습니다.

> **💡 참고**
> 
> 원격 애플리케이션에 접근하기 위해 프록시를 사용해야 하는 경우 `spring.devtools.remote.proxy.host`와 `spring.devtools.remote.proxy.port` 프로퍼티를 구성하세요.

### 원격 업데이트

원격 클라이언트는 로컬 재시작과 동일한 방식으로 애플리케이션 클래스패스의 변경사항을 모니터링합니다. 업데이트된 리소스는 원격 애플리케이션으로 푸시되고 (필요한 경우) 재시작을 트리거합니다. 이는 로컬에서 사용할 수 없는 클라우드 서비스를 사용하는 기능을 반복 작업할 때 유용할 수 있습니다. 일반적으로 원격 업데이트와 재시작은 전체 빌드 및 배포 주기보다 훨씬 빠릅니다.

개발 환경이 느린 경우, 대기 시간이 충분하지 않아 클래스의 변경사항이 배치로 나뉠 수 있습니다. 첫 번째 배치의 클래스 변경사항이 업로드된 후 서버가 재시작됩니다. 서버가 재시작 중이므로 다음 배치를 애플리케이션에 전송할 수 없습니다.

이는 일반적으로 일부 클래스를 업로드하지 못했다는 RemoteSpringApplication 로그의 경고와 그에 따른 재시도로 나타납니다. 하지만 첫 번째 배치의 변경사항이 업로드된 후 애플리케이션 코드의 불일치와 재시작 실패로 이어질 수도 있습니다. 이러한 문제가 지속적으로 발생하면, 개발 환경에 맞는 값으로 `spring.devtools.restart.poll-interval`과 `spring.devtools.restart.quiet-period` 매개변수를 증가시켜 보세요. 이러한 프로퍼티 구성에 대한 자세한 내용은 [File System Watcher 구성하기](#file-system-watcher-구성하기) 섹션을 참조하세요.

> **💡 참고**
> 
> 원격 클라이언트가 실행 중일 때만 파일이 모니터링됩니다. 원격 클라이언트를 시작하기 전에 파일을 변경하면 원격 서버로 푸시되지 않습니다.

---