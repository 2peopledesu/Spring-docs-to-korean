---
title: SpringApplication
weight: 1
date: 2025-02-22
---

## SpringApplication

[SpringApplication](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/SpringApplication.html) 클래스는 `main()` 메서드에서 시작되는 Spring 애플리케이션을 부트스트랩하는 편리한 방법을 제공합니다. 대부분의 경우 다음 예제와 같이 정적 [SpringApplication.run(Class, String…​)](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/SpringApplication.html#run(java.lang.Class,java.lang.String%E2%80%A6%E2%80%8B)) 메서드에 위임할 수 있습니다:

**Java**
```java
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

**Kotlin**
```kotlin
@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
    runApplication<MyApplication>(*args)
}
```

애플리케이션이 시작되면 다음과 유사한 출력이 표시됩니다:

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.4.3)

2025-02-20T14:16:02.993Z  INFO 126323 --- [           main] o.s.b.d.f.logexample.MyApplication       : Starting MyApplication using Java 17.0.14 with PID 126323 (/opt/apps/myapp.jar started by myuser in /opt/apps/)
2025-02-20T14:16:03.006Z  INFO 126323 --- [           main] o.s.b.d.f.logexample.MyApplication       : No active profile set, falling back to 1 default profile: "default"
2025-02-20T14:16:07.963Z  INFO 126323 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8080 (http)
2025-02-20T14:16:08.040Z  INFO 126323 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2025-02-20T14:16:08.041Z  INFO 126323 --- [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.36]
2025-02-20T14:16:08.241Z  INFO 126323 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2025-02-20T14:16:08.262Z  INFO 126323 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 4960 ms
2025-02-20T14:16:09.633Z  INFO 126323 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2025-02-20T14:16:09.670Z  INFO 126323 --- [           main] o.s.b.d.f.logexample.MyApplication       : Started MyApplication in 8.489 seconds (process running for 9.432)
2025-02-20T14:16:09.677Z  INFO 126323 --- [ionShutdownHook] o.s.b.w.e.tomcat.GracefulShutdown        : Commencing graceful shutdown. Waiting for active requests to complete
2025-02-20T14:16:09.683Z  INFO 126323 --- [tomcat-shutdown] o.s.b.w.e.tomcat.GracefulShutdown        : Graceful shutdown complete
```

기본적으로 `INFO` 로깅 메시지가 표시되며, 애플리케이션을 실행한 사용자와 같은 관련 시작 세부 정보가 포함됩니다. `INFO` 이외의 로그 레벨이 필요한 경우 [로그 레벨](../logging#features.logging.log-levels)에 설명된 대로 설정할 수 있습니다. 애플리케이션 버전은 메인 애플리케이션 클래스의 패키지에서 구현 버전을 사용하여 결정됩니다. 시작 정보 로깅은 `spring.main.log-startup-info`를 `false`로 설정하여 끌 수 있습니다. 이렇게 하면 애플리케이션의 활성 프로파일 로깅도 꺼집니다.

> **💡 참고**
> 
> 시작 시 추가 로깅을 하려면 [SpringApplication](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/SpringApplication.html)의 하위 클래스에서 `logStartupInfo(boolean)`를 재정의할 수 있습니다.

## Startup Failure

애플리케이션 시작에 실패하면, 등록된 [FailureAnalyzer](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/diagnostics/FailureAnalyzer.html) bean들이 전용 오류 메시지와 문제를 해결하기 위한 구체적인 조치를 제공할 기회를 얻습니다. 예를 들어, 포트 `8080`에서 웹 애플리케이션을 시작하는데 해당 포트가 이미 사용 중인 경우 다음과 유사한 메시지가 표시됩니다:

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that is listening on port 8080 or configure this application to listen on another port.
```

> **💡 참고**
> 
> Spring Boot는 수많은 [FailureAnalyzer](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/diagnostics/FailureAnalyzer.html) 구현을 제공하며, [직접 추가](../../how-to/application#howto.application.failure-analyzer)할 수도 있습니다.

실패 분석기가 예외를 처리할 수 없는 경우에도 무엇이 잘못되었는지 더 잘 이해하기 위해 전체 조건 보고서를 표시할 수 있습니다. 이를 위해서는 [debug 프로퍼티를 활성화](../external-config)하거나 [ConditionEvaluationReportLoggingListener](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/logging/ConditionEvaluationReportLoggingListener.html)에 대해 [DEBUG 로깅을 활성화](../logging#features.logging.log-levels)해야 합니다.

예를 들어, `java -jar`를 사용하여 애플리케이션을 실행하는 경우 다음과 같이 `debug` 프로퍼티를 활성화할 수 있습니다:

```shell
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

## Lazy Initialization

[SpringApplication](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/SpringApplication.html)은 애플리케이션이 지연 초기화되도록 허용합니다. 지연 초기화가 활성화되면, bean들은 애플리케이션 시작 시가 아닌 필요할 때 생성됩니다. 결과적으로 지연 초기화를 활성화하면 애플리케이션 시작에 걸리는 시간을 줄일 수 있습니다. 웹 애플리케이션에서 지연 초기화를 활성화하면 HTTP 요청을 받을 때까지 많은 웹 관련 빈들이 초기화되지 않습니다.

지연 초기화의 단점은 애플리케이션의 문제점 발견이 지연될 수 있다는 것입니다. 잘못 구성된 bean이 지연 초기화되면 시작 시에는 실패가 발생하지 않고 bean이 초기화될 때만 문제가 드러나게 됩니다. 또한 JVM이 시작 시 초기화되는 빈들뿐만 아니라 애플리케이션의 모든 빈들을 수용할 수 있는 충분한 메모리를 가지고 있는지 확인하는 데도 주의를 기울여야 합니다. 이러한 이유로 지연 초기화는 기본적으로 활성화되어 있지 않으며, 지연 초기화를 활성화하기 전에 JVM의 힙 크기를 세밀하게 조정하는 것이 권장됩니다.

지연 초기화는 `SpringApplicationBuilder`의 `lazyInitialization` 메서드나 [`SpringApplication`](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/builder/SpringApplicationBuilder.html)의 `setLazyInitialization` 메서드를 사용하여 프로그래밍 방식으로 활성화할 수 있습니다. 또는 다음 예제와 같이 `spring.main.lazy-initialization` 프로퍼티를 사용하여 활성화할 수 있습니다:

**Properties**
```properties
spring.main.lazy-initialization=true
```

**YAML**
```yaml
spring:
  main:
    lazy-initialization: true
```

> **💡 참고**
> 
> 애플리케이션의 나머지 부분에 대해 지연 초기화를 사용하면서 특정 빈들에 대해 지연 초기화를 비활성화하려면 `@Lazy(false)` 어노테이션을 사용하여 해당 빈들의 지연 속성을 명시적으로 false로 설정할 수 있습니다.

## Customizing the Banner

시작 시 출력되는 배너는 클래스패스에 `banner.txt` 파일을 추가하거나 `spring.banner.location` 프로퍼티를 해당 파일의 위치로 설정하여 변경할 수 있습니다. 파일의 인코딩이 UTF-8이 아닌 경우 `spring.banner.charset`을 설정할 수 있습니다.

`banner.txt` 파일 내에서는 [Environment](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/core/env/Environment.html)에서 사용 가능한 모든 Key와 다음 placeholders를 사용할 수 있습니다:

**표 1. 배너 변수**

| 변수 | 설명 |
|------|------|
| ${application.version} | MANIFEST.MF에 선언된 애플리케이션의 버전 번호입니다. 예: Implementation-Version: 1.0은 1.0으로 출력됩니다. |
| ${application.formatted-version} | MANIFEST.MF에 선언된 애플리케이션의 버전 번호를 표시용으로 형식화한 것입니다(대괄호로 둘러싸고 v 접두사 추가). 예: (v1.0) |
| ${spring-boot.version} | 사용 중인 Spring Boot 버전입니다. 예: 3.4.3 |
| ${spring-boot.formatted-version} | 사용 중인 Spring Boot 버전을 표시용으로 형식화한 것입니다(대괄호로 둘러싸고 v 접두사 추가). 예: (v3.4.3) |
| ${Ansi.NAME} (또는 ${AnsiColor.NAME}, ${AnsiBackground.NAME}, ${AnsiStyle.NAME}) | NAME은 ANSI 이스케이프 코드의 이름입니다. 자세한 내용은 [AnsiPropertySource](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ansi/AnsiPropertySource.html)를 참조하세요. |
| ${application.title} | MANIFEST.MF에 선언된 애플리케이션의 제목입니다. 예: Implementation-Title: MyApp은 MyApp으로 출력됩니다. |

> **💡 참고**
> 
> 프로그래밍 방식으로 배너를 생성하려면 `SpringApplication.setBanner(…​)` 메서드를 사용할 수 있습니다. [Banner](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/Banner.html) 인터페이스를 사용하고 자신만의 `printBanner()` 메서드를 구현하세요.

`spring.main.banner-mode` 프로퍼티를 사용하여 배너를 System.out(`console`)에 출력할지, 구성된 로거(`log`)로 보낼지, 아니면 전혀 생성하지 않을지(`off`) 결정할 수 있습니다.

출력된 배너는 다음 이름으로 싱글톤 빈으로 등록됩니다: `springBootBanner`.

> **💡 참고**
> 
> `application.title`, `application.version`, `application.formatted-version` 프로퍼티는 Spring Boot 런처와 함께 `java -jar` 또는 `java -cp`를 사용하는 경우에만 사용할 수 있습니다. 압축 해제된 jar를 실행하고 `java -cp <classpath> <mainclass>`로 시작하거나 네이티브 이미지로 애플리케이션을 실행하는 경우에는 값이 해석되지 않습니다.
>
> `application.` 프로퍼티를 사용하려면 `java -jar`를 사용하여 패키지된 jar로 애플리케이션을 실행하거나 `java org.springframework.boot.loader.launch.JarLauncher`를 사용하여 압축 해제된 jar로 실행하세요. 이렇게 하면 클래스패스를 빌드하고 앱을 실행하기 전에 `application.` 배너 프로퍼티가 초기화됩니다.

## Customizing SpringApplication

SpringApplication의 기본값이 마음에 들지 않는다면 로컬 인스턴스를 생성하고 커스터마이즈할 수 있습니다. 예를 들어, 배너를 끄려면 다음과 같이 작성할 수 있습니다:

**Java**
```java
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(MyApplication.class);
        application.setBannerMode(Banner.Mode.OFF);
        application.run(args);
    }

}
```

**Kotlin**
```kotlin
@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
    runApplication<MyApplication>(*args) {
        setBannerMode(Banner.Mode.OFF)
    }
}
```

> **💡 참고**
> 
> [SpringApplication](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/SpringApplication.html)에 전달되는 생성자 인수는 Spring 빈의 구성 소스입니다. 대부분의 경우 [@Configuration](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html) 클래스에 대한 참조이지만, [@Component](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html) 클래스에 대한 직접 참조일 수도 있습니다.

`application.properties` 파일을 사용하여 SpringApplication을 구성할 수도 있습니다. 자세한 내용은 [외부화된 구성](../externalized-configuration)을 참조하세요.

구성 옵션의 전체 목록은 [SpringApplication API 문서](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/SpringApplication.html)를 참조하세요.

## Fluent Builder API

ApplicationContext 계층 구조(부모/자식 관계가 있는 여러 컨텍스트)를 구축해야 하거나 유동적 빌더 API를 선호하는 경우 SpringApplicationBuilder를 사용할 수 있습니다.

SpringApplicationBuilder를 사용하면 다음 예제와 같이 여러 메서드 호출을 연결할 수 있고 계층 구조를 만들 수 있는 `parent`와 `child` 메서드를 포함합니다:

**Java**
```java
new SpringApplicationBuilder()
    .sources(Parent.class)
    .child(Application.class)
    .bannerMode(Banner.Mode.OFF)
    .run(args);
```

**Kotlin**
```kotlin
SpringApplicationBuilder()
    .sources(Parent::class.java)
    .child(Application::class.java)
    .bannerMode(Banner.Mode.OFF)
    .run(*args)
```

> **💡 참고**
> 
> [ApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html) 계층 구조를 만들 때 몇 가지 제한 사항이 있습니다. 예를 들어, 웹 컴포넌트는 반드시 자식 컨텍스트 내에 포함되어야 하며, 부모와 자식 컨텍스트 모두에 동일한 [Environment](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/Environment.html)가 사용됩니다. 자세한 내용은 [SpringApplicationBuilder API 문서](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/builder/SpringApplicationBuilder.html)를 참조하세요.

## Application Availability

플랫폼에 배포될 때 애플리케이션은 [Kubernetes Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)와 같은 인프라를 사용하여 플랫폼에 가용성 정보를 제공할 수 있습니다. Spring Boot는 일반적으로 사용되는 "liveness"와 "readiness" 가용성 상태에 대한 기본 지원을 포함합니다. Spring Boot의 "actuator" 지원을 사용하는 경우 이러한 상태는 상태 엔드포인트 그룹으로 노출됩니다.

또한 [ApplicationAvailability](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/availability/ApplicationAvailability.html) 인터페이스를 자신의 빈에 주입하여 가용성 상태를 얻을 수도 있습니다.

### Liveness State

애플리케이션의 "Liveness" 상태는 내부 상태가 올바르게 작동하거나 현재 실패한 경우 스스로 복구할 수 있는지를 알려줍니다. "Liveness" 상태가 손상되었다는 것은 애플리케이션이 복구할 수 없는 상태에 있으며 인프라가 애플리케이션을 재시작해야 한다는 것을 의미합니다.

> **💡 참고**
> 
> 일반적으로 "Liveness" 상태는 [health checks](../../actuator/endpoints#actuator.endpoints.health)과 같은 외부 점검을 기반으로 해서는 안 됩니다. 그렇게 하면 실패한 외부 시스템(데이터베이스, 웹 API, 외부 캐시)이 플랫폼 전체에서 대규모 재시작과 연쇄 실패를 유발할 수 있습니다.

Spring Boot 애플리케이션의 내부 상태는 주로 Spring ApplicationContext로 표현됩니다. 애플리케이션 컨텍스트가 성공적으로 시작되면 Spring Boot는 애플리케이션이 유효한 상태에 있다고 가정합니다. 컨텍스트가 새로 고쳐지면 애플리케이션은 즉시 활성 상태로 간주됩니다. [Application Events and Listeners](#Application-Events-and-Listeners)를 참조하세요.

### Readiness State

애플리케이션의 "Readiness" 상태는 애플리케이션이 트래픽을 처리할 준비가 되었는지를 알려줍니다. "Readiness" 상태가 실패하면 플랫폼에 현재 애플리케이션으로 트래픽을 라우팅하지 말라고 알립니다. 이는 일반적으로 시작 중에 [CommandLineRunner](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/CommandLineRunner.html)와 [ApplicationRunner](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/ApplicationRunner.html) 컴포넌트가 처리되는 동안, 또는 애플리케이션이 추가 트래픽을 처리하기에 너무 바쁘다고 판단할 때 발생합니다.

애플리케이션과 명령줄 러너가 호출되면 애플리케이션은 준비된 것으로 간주됩니다. [Application Events and Listeners](#Application-Events-and-Listeners)를 참조하세요.

> **💡 참고**
> 
> 시작 중에 실행되어야 하는 작업은 [@PostConstruct](https://jakarta.ee/specifications/annotations/2.1/apidocs/jakarta/annotation/PostConstruct.html)와 같은 Spring 컴포넌트 수명 주기 콜백을 사용하는 대신 [CommandLineRunner](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/CommandLineRunner.html)와 [ApplicationRunner](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationRunner.html) 컴포넌트로 실행되어야 합니다.

### Managing the Application Availability State

애플리케이션 컴포넌트는 [ApplicationAvailability](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/availability/ApplicationAvailability.html) 인터페이스를 주입하고 이를 호출하여 언제든지 현재 가용성 상태를 검색할 수 있습니다. 더 자주 애플리케이션은 상태 업데이트를 수신하거나 애플리케이션의 상태를 업데이트하고 싶을 것입니다.

예를 들어, Kubernetes "exec Probe"가 이 파일을 볼 수 있도록 애플리케이션의 "Readiness" 상태를 파일로 내보낼 수 있습니다:

**Java**
```java
@Component
public class MyReadinessStateExporter {

    @EventListener
    public void onStateChange(AvailabilityChangeEvent<ReadinessState> event) {
        switch (event.getState()) {
            case ACCEPTING_TRAFFIC -> {
                // Create /tmp/healthy
            }
            case REFUSING_TRAFFIC -> {
                // Remove /tmp/healthy
            }
        }
    }

}
```

**Kotlin**
```kotlin
@Component
class MyReadinessStateExporter {

    @EventListener
    fun onStateChange(event: AvailabilityChangeEvent<ReadinessState?>) {
        when (event.state) {
            ReadinessState.ACCEPTING_TRAFFIC -> {
                // Create /tmp/healthy
            }
            ReadinessState.REFUSING_TRAFFIC -> {
                // Remove /tmp/healthy 파일 제거
            }
            else -> {
                // ...
            }
        }
    }

}
```

또한 애플리케이션이 손상되고 복구할 수 없을 때 애플리케이션의 상태를 업데이트할 수도 있습니다:

**Java**
```java

@Component
public class MyLocalCacheVerifier {

    private final ApplicationEventPublisher eventPublisher;

    public MyLocalCacheVerifier(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    public void checkLocalCache() {
        try {
            // ...
        }
        catch (CacheCompletelyBrokenException ex) {
            AvailabilityChangeEvent.publish(this.eventPublisher, ex, LivenessState.BROKEN);
        }
    }

}
```

**Kotlin**
```kotlin

@Component
class MyLocalCacheVerifier(private val eventPublisher: ApplicationEventPublisher) {

    fun checkLocalCache() {
        try {
            // ...
        } catch (ex: CacheCompletelyBrokenException) {
            AvailabilityChangeEvent.publish(eventPublisher, ex, LivenessState.BROKEN)
        }
    }

}
```

Spring Boot는 [Actuator Health Endpoints를 통해 "Liveness"와 "Readiness"를 위한 Kubernetes HTTP 프로브](../actuator/endpoints#actuator.endpoints.kubernetes-probes)를 제공합니다. Kubernetes에 Spring Boot 애플리케이션을 배포하는 것에 대한 자세한 안내는 [ deploying Spring Boot applications on Kubernetes in the dedicated section](../../how-to/deployment/cloud#kubernetes)를 참고하세요.

## Application Events and Listeners

일반적인 Spring Framework 이벤트는 [ContextRefreshedEvent](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html), SpringApplication 외에도 몇 가지 추가적인 애플리케이션 이벤트를 보냅니다.

> **💡 참고**
> 
> 일부 이벤트는 실제로 [ApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html)가 생성되기 전에 트리거되므로 [@Bean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)으로 리스너를 등록할 수 없습니다. `SpringApplication.addListeners(…​)` 메서드나 `SpringApplicationBuilder.listeners(…​)` 메서드를 사용하여 등록할 수 있습니다. 애플리케이션이 생성되는 방식에 관계없이 해당 리스너가 자동으로 등록되기를 원한다면 프로젝트에 `META-INF/spring.factories` 파일을 추가하고 [ApplicationListener](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/ApplicationListener.html)key를 다음 아래 예시와 같이 제출하고 사용할 수 있습니다.
>
>```
>org.springframework.context.ApplicationListener=com.example.project.MyListener
>```

애플리케이션이 실행될 때 이벤트는 다음 순서로 전송됩니다:

1. `ApplicationStartingEvent`는 실행 시작 시점에 전송되지만, 리스너와 초기화 프로그램 등록을 제외한 어떤 처리도 하기 전에 전송됩니다.

2. `ApplicationEnvironmentPreparedEvent`는 컨텍스트에서 사용할 Environment가 준비되었지만 컨텍스트가 생성되기 전에 전송됩니다.

3. `ApplicationContextInitializedEvent`는 ApplicationContext가 준비되고 ApplicationContextInitializers가 호출되었지만 빈 정의가 로드되기 전에 전송됩니다.

4. `ApplicationPreparedEvent`는 빈 정의가 로드된 후 리프레시가 시작되기 직전에 전송됩니다.

5. `ApplicationStartedEvent`는 컨텍스트가 리프레시된 후  application and command-line runners가 호출되기 전에 전송됩니다.

6. `AvailabilityChangeEvent`는 애플리케이션이 실행 중인 것으로 간주됨을 나타내기 위해 `LivenessState.CORRECT`와 함께 바로 전송됩니다.

7. `ApplicationReadyEvent`는  application and command-line runners가 호출된 후에 전송됩니다.

8. `AvailabilityChangeEvent`는 애플리케이션이 요청을 처리할 준비가 되었음을 나타내기 위해 `ReadinessState.ACCEPTING_TRAFFIC`와 함께 바로 전송됩니다.

9. `ApplicationFailedEvent`는 시작 중 예외가 발생하면 전송됩니다.

위 목록은 SpringApplication에 연결된 `SpringApplicationEvent`만 포함합니다. 이 외에도 `ApplicationPreparedEvent` 이후와 `ApplicationStartedEvent` 이전에 다음 이벤트들이 발행됩니다:

- WebServer가 준비된 후 `WebServerInitializedEvent`가 전송됩니다. `ServletWebServerInitializedEvent`와 `ReactiveWebServerInitializedEvent`는 각각 서블릿과 리액티브 변형입니다.
- `ApplicationContext`가 리프레시될 때 `ContextRefreshedEvent`가 전송됩니다.

> **💡 참고**
> 
> 애플리케이션 이벤트를 사용할 필요는 없지만, 이벤트가 존재한다는 것을 알아두면 유용할 수 있습니다. 내부적으로 Spring Boot는 다양한 작업을 처리하기 위해 이벤트를 사용합니다.

> **💡 참고**
> 
> 이벤트 리스너는 기본적으로 동일한 스레드에서 실행되므로 시간이 오래 걸리는 작업을 실행해서는 안 됩니다. 대신 애플리케이션과 커맨드라인 러너를 사용하는 것을 고려하세요.

애플리케이션 이벤트는 Spring Framework의 이벤트 발행 메커니즘을 사용하여 전송됩니다. 이 메커니즘의 일부는 자식 컨텍스트의 리스너에게 발행된 이벤트가 모든 상위 컨텍스트의 리스너에게도 발행되도록 합니다. 결과적으로 애플리케이션이 SpringApplication 인스턴스의 계층 구조를 사용하는 경우, 리스너는 동일한 유형의 애플리케이션 이벤트의 여러 인스턴스를 수신할 수 있습니다.

리스너가 자신의 컨텍스트에 대한 이벤트와 하위 컨텍스트에 대한 이벤트를 구분할 수 있도록 하려면, 애플리케이션 컨텍스트를 주입하도록 요청하고 주입된 컨텍스트를 이벤트의 컨텍스트와 비교해야 합니다. 컨텍스트는 `ApplicationContextAware`를 구현하거나 리스너가 빈인 경우 `@Autowired`를 사용하여 주입할 수 있습니다.

## Web Environment

SpringApplication은 사용자를 대신하여 올바른 유형의 ApplicationContext를 생성하려고 시도합니다. [WebApplicationType](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/WebApplicationType.html)을 결정하는 알고리즘은 다음과 같습니다:

- Spring MVC가 있으면 [`AnnotationConfigServletWebServerApplicationContext`](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/web/servlet/context/AnnotationConfigServletWebServerApplicationContext.html)가 사용됩니다
- Spring MVC가 없고 Spring WebFlux가 있으면 [`AnnotationConfigReactiveWebServerApplicationContext`](https://docs.spring.io/spring-boot/3.4.3/api/java/org/springframework/boot/web/reactive/context/AnnotationConfigReactiveWebServerApplicationContext.html)가 사용됩니다
- 그 외의 경우 [`AnnotationConfigApplicationContext`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/AnnotationConfigApplicationContext.html)가 사용됩니다

이는 Spring MVC와 Spring WebFlux의 새로운 [WebClient](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html)를 동일한 애플리케이션에서 사용하는 경우 기본적으로 Spring MVC가 사용된다는 것을 의미합니다. `setWebApplicationType(WebApplicationType)`을 호출하여 쉽게 이를 재정의할 수 있습니다.

`setApplicationContextFactory(…​)`를 호출하여 사용되는 ApplicationContext 유형을 완전히 제어하는 것도 가능합니다.

> **💡 참고**
> 
> JUnit 테스트 내에서 SpringApplication을 사용할 때는 `setWebApplicationType(WebApplicationType.NONE)`을 호출하는 것이 바람직한 경우가 많습니다.

## Accessing Application Arguments

`SpringApplication.run(…​)`에 전달된 애플리케이션 인수에 접근해야 하는 경우 `ApplicationArguments` 빈을 주입할 수 있습니다. ApplicationArguments 인터페이스는 원시 `String[]` 인수뿐만 아니라 파싱된 `option`과 `non-option`에 대한 접근도 제공합니다. 다음 예제를 참조하세요:

**Java**
```java
@Component
public class MyBean {

    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        if (debug) {
            System.out.println(files);
        }
        // "--debug logfile.txt"로 실행하면 ["logfile.txt"]가 출력됩니다
    }

}
```

**Kotlin**
```kotlin
@Component
class MyBean(args: ApplicationArguments) {

    init {
        val debug = args.containsOption("debug")
        val files = args.nonOptionArgs
        if (debug) {
            println(files)
        }
        // "--debug logfile.txt"로 실행하면 ["logfile.txt"]가 출력됩니다
    }

}
```

> **💡 참고**
> 
> Spring Boot는 Spring Environment에 [CommandLinePropertySource](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/core/env/CommandLinePropertySource.html)도 등록합니다. 이를 통해 `@Value` 어노테이션을 사용하여 단일 애플리케이션 인수를 주입할 수도 있습니다.

## Using the ApplicationRunner or CommandLineRunner

SpringApplication이 시작된 후 특정 코드를 실행해야 하는 경우 `ApplicationRunner` 또는 `CommandLineRunner` 인터페이스를 구현할 수 있습니다. 두 인터페이스는 동일한 방식으로 작동하며 `SpringApplication.run(…​)`이 완료되기 직전에 호출되는 단일 `run` 메서드를 제공합니다.

> **💡 참고**
> 
> 이 계약은 애플리케이션 시작 후 트래픽을 받기 전에 실행되어야 하는 작업에 적합합니다.

[`CommandLineRunner`](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/CommandLineRunner.html) 인터페이스는 애플리케이션 인수를 문자열 배열로 제공하는 반면, ApplicationRunner는 앞서 설명한 ApplicationArguments 인터페이스를 사용합니다. 다음 예제는 `run` 메서드가 있는 CommandLineRunner를 보여줍니다:

**Java**
```java
@Component
public class MyCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        // 작업 수행...
    }

}
```

**Kotlin**
```kotlin

@Component
class MyCommandLineRunner : CommandLineRunner {

    override fun run(vararg args: String) {
        // 작업 수행...
    }

}
```

특정 순서로 호출되어야 하는 여러 `CommandLineRunner` 또는 `ApplicationRunner` 빈이 정의된 경우 추가로 `Ordered` 인터페이스를 구현하거나 `@Order` 어노테이션을 사용할 수 있습니다.

## Application Exit

각 `SpringApplication`은 `ApplicationContext`가 정상적으로 종료되도록 JVM에 셧다운 훅을 등록합니다. 모든 표준 Spring 수명 주기 콜백(`DisposableBean` 인터페이스나 `@PreDestroy` 어노테이션 등)을 사용할 수 있습니다.

또한 빈은 `SpringApplication.exit()`가 호출될 때 특정 종료 코드를 반환하고 싶은 경우 `ExitCodeGenerator` 인터페이스를 구현할 수 있습니다. 이 종료 코드는 `System.exit()`에 전달되어 상태 코드로 반환될 수 있습니다. 다음 예제를 참조하세요:

**Java**
```java
@SpringBootApplication
public class MyApplication {

    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
        return () -> 42;
    }

    public static void main(String[] args) {
        System.exit(SpringApplication.exit(SpringApplication.run(MyApplication.class, args)));
    }

}
```

**Kotlin**
```kotlin
@SpringBootApplication
class MyApplication {

    @Bean
    fun exitCodeGenerator() = ExitCodeGenerator { 42 }

}

fun main(args: Array<String>) {
    exitProcess(SpringApplication.exit(runApplication<MyApplication>(*args)))
}
```

또한 `ExitCodeGenerator` 인터페이스는 예외에 의해 구현될 수 있습니다. 이러한 예외가 발생하면 Spring Boot는 구현된 `getExitCode()` 메서드가 제공하는 종료 코드를 반환합니다.

`ExitCodeGenerator`가 여러 개 있는 경우, 생성된 첫 번째 0이 아닌 종료 코드가 사용됩니다. 생성기가 호출되는 순서를 제어하려면 추가로 `Ordered` 인터페이스를 구현하거나 `@Order` 어노테이션을 사용하세요.

## Admin Features

`spring.application.admin.enabled` 프로퍼티를 지정하여 애플리케이션의 관리자 관련 기능을 활성화할 수 있습니다. 이는 플랫폼 MBeanServer에 [`SpringApplicationAdminMXBean`](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/admin/SpringApplicationAdminMXBean.html)을 노출합니다. 이 기능을 사용하여 Spring Boot 애플리케이션을 원격으로 관리할 수 있습니다. 이 기능은 서비스 wrapper 구현에도 유용할 수 있습니다.

> **💡 참고**
> 
> 애플리케이션이 실행 중인 HTTP 포트를 알고 싶다면 `local.server.port` 키로 프로퍼티를 가져오세요.

## Application Startup tracking

애플리케이션 시작 중에 `SpringApplication`과 `ApplicationContext`는 애플리케이션 수명 주기, 빈 수명 주기 또는 애플리케이션 이벤트 처리와 관련된 많은 작업을 수행합니다. `ApplicationStartup`을 사용하면 Spring Framework가 StartupStep 객체로 애플리케이션 시작 순서를 추적할 수 있습니다. 이 데이터는 프로파일링 목적으로 수집하거나 애플리케이션 시작 프로세스를 더 잘 이해하기 위해 사용할 수 있습니다.

`SpringApplication` 인스턴스를 설정할 때 `ApplicationStartup` 구현을 선택할 수 있습니다. 예를 들어, `BufferingApplicationStartup`을 사용하려면 다음과 같이 작성할 수 있습니다:

**Java**
```java
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApplication.class);
        app.setApplicationStartup(new BufferingApplicationStartup(2048));
        app.run(args);
    }

}
```

**Kotlin**
```kotlin
@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
    runApplication<MyApplication>(*args) {
        applicationStartup = BufferingApplicationStartup(2048)
    }
}
```

첫 번째로 사용 가능한 구현인 [`FlightRecorderApplicationStartup`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/core/metrics/jfr/FlightRecorderApplicationStartup.html)은 Spring Framework에서 제공됩니다. 이는 Spring 특정 시작 이벤트를 Java Flight Recorder 세션에 추가하며 애플리케이션을 프로파일링하고 Spring 컨텍스트 수명 주기를 JVM 이벤트(할당, GC, 클래스 로딩 등)와 연관시키기 위한 것입니다. 구성이 완료되면 Flight Recorder를 활성화하여 데이터를 기록할 수 있습니다:

```bash
$ java -XX:StartFlightRecording:filename=recording.jfr,duration=10s -jar demo.jar
```

Spring Boot는 `BufferingApplicationStartup` 변형과 함께 제공됩니다. 이 구현은 시작 단계를 버퍼링하고 외부 메트릭 시스템으로 드레이닝하기 위한 것입니다. 애플리케이션은 모든 컴포넌트에서 `BufferingApplicationStartup` 유형의 빈을 요청할 수 있습니다.

Spring Boot는 이 정보를 JSON 문서로 제공하는 [시작 엔드포인트](../../api/rest/actuator/startup)를 노출하도록 구성할 수도 있습니다.

## Virtual threads

Java 21 이상에서 실행 중인 경우 `spring.threads.virtual.enabled` 프로퍼티를 `true`로 설정하여 가상 스레드를 활성화할 수 있습니다.
 
애플리케이션에 이 옵션을 활성화하기 전에 [공식 Java 가상 스레드 문서](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html)를 읽어보는 것이 좋습니다. 경우에 따라 "고정된 가상 스레드" 때문에 애플리케이션의 처리량이 낮아질 수 있습니다. 이 페이지에서는 JDK Flight Recorder나 `jcmd` CLI를 사용하여 이러한 경우를 감지하는 방법도 설명합니다.

> **💡 참고**
> 
> 가상 스레드가 활성화되면 스레드 풀을 구성하는 프로퍼티는 더 이상 효과가 없습니다. 가상 스레드는 전용 스레드 풀이 아닌 JVM 전체 플랫폼 스레드 풀에서 스케줄링되기 때문입니다.

> **⚠️ 경고**
>가상 스레드의 한 가지 부작용은 데몬 스레드라는 것입니다. JVM은 모든 스레드가 데몬 스레드인 경우 종료됩니다. 이 동작은 예를 들어 `@Scheduled` 빈을 사용하여 애플리케이션을 실행 상태로 유지하는 경우 문제가 될 수 있습니다. 가상 스레드를 사용하는 경우 스케줄러 스레드, 데몬 스레드는 가상 스레드이며 JVM을 실행 상태로 유지하지 않습니다. 이는 스케줄링에만 영향을 미치는 것이 아니라 다른 기술에서도 발생할 수 있습니다. 모든 경우에 JVM을 실행 상태로 유지하려면 `spring.main.keep-alive` 프로퍼티를 `true`로 설정하는 것이 좋습니다. 이렇게 하면 모든 스레드가 가상 스레드인 경우에도 JVM이 실행 상태를 유지합니다.