---
title: Spring Beans and Dependency Injection
date: 2024-02-21
weight: 6
---

## Spring Beans와 의존성 주입

빈을 정의하고 의존성을 주입하기 위해 Spring Framework의 표준 기술을 자유롭게 사용할 수 있습니다. 일반적으로 의존성을 연결하기 위해 생성자 주입을 사용하고, 빈을 찾기 위해 [@ComponentScan](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/context/annotation/ComponentScan.html)을 사용하는 것을 권장합니다.

위에서 제안한 대로 코드를 구성하면(애플리케이션 클래스를 최상위 패키지에 위치시키면), 인자 없이 @ComponentScan을 추가하거나 이를 암시적으로 포함하는 [@SpringBootApplication](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html) 어노테이션을 사용할 수 있습니다. 모든 애플리케이션 컴포넌트([@Component](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/stereotype/Component.html), [@Service](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/stereotype/Service.html), [@Repository](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/stereotype/Repository.html), [@Controller](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/stereotype/Controller.html) 등)는 자동으로 Spring Bean으로 등록됩니다.

다음 예제는 필수 `RiskAssessor` 빈을 얻기 위해 생성자 주입을 사용하는 @Service Bean을 보여줍니다:

```java
import org.springframework.stereotype.Service;

@Service
public class MyAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public MyAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...
}
```

빈에 생성자가 둘 이상 있는 경우, Spring이 사용할 생성자에 [@Autowired](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/beans/factory/annotation/Autowired.html)를 표시해야 합니다:

```java
import java.io.PrintStream;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class MyAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    private final PrintStream out;

    @Autowired
    public MyAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
        this.out = System.out;
    }

    public MyAccountService(RiskAssessor riskAssessor, PrintStream out) {
        this.riskAssessor = riskAssessor;
        this.out = out;
    }

    // ...
}
```

생성자 주입을 사용하면 `riskAssessor` 필드를 final로 표시할 수 있어, 이후에 변경할 수 없음을 나타낼 수 있습니다.

---