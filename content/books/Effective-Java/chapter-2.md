---
title: Creating and Destroying Objects
weight: 1
sidebar:
  open: false
---
# 객체의 생성과 소멸

이 장은 객체의 생성과 소멸을 다룹니다. 
언제, 어떻게 객체를 생성하고, 생성하지 않을지, 객체가 제때 소멸되도록 보장하는 방법, 소멸 전 필요한 정리 작업을 관리하는 방법 등을 설명합니다.

## ITEM 1 : Consider static factory methods instead of constructors
### 5P
#### 핵심 아이디어
클래스 인스턴스 생성 시 public 생성자 대신 **정적 팩토리 메서드**를 고려해야합니다.

##### 정적 팩토리 메서드 예시
```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

> boolean 기본 타입을 Boolean 객체로 변환하며, 새로운 객체를 생성하지 않고 기존 인스턴스를 재사용합니다.

#### 정적 팩토리 메서드의 장점

##### 1. 이름을 가진 생성자

```java

// 생성자: 동일한 시그니처 구분 불가
public Product(int type) { ... }

// 정적 팩토리: 목적에 따라 이름 지정 가능
public static Product createFoodProduct() { ... }
public static Product createElectronicsProduct() { ... }
```
##### 2. 인스턴스 생성 통제

```java

public class Product {
    private static final Product INSTANCE = new Product();
    
    // 외부에서 new Product() 금지
    private Product() {}
    
    public static Product getInstance() {
        return INSTANCE; // 항상 동일 인스턴스 반환
    }
}
```

##### 3. 하위 타입 반환 가능

``` java

public interface Product {
    static Product create(String type) {
        if (type.equals("food")) return new Food();
        else return new Electronics();
    }
}
```

##### 4. 유연한 반환 타입

```java

// Java의 Collections.emptyList()
public static <T> List<T> emptyList() {
    return (List<T>) EMPTY_LIST;
}
```

#### 정적 팩토리 메서드의 단점

##### 1. 상속 불가

- public 또는 protected 생성자가 없는 클래스는 상속할 수 없습니다.

```java

public class UtilityClass {
    private UtilityClass() {} // 상속 불가능
    public static void doSomething() { ... }
}
```

##### 2. 문서화의 어려움
- API 문서에서 정적 팩토리 메서드를 찾기 어려울 수 있음
- 생성자와 달리 별도의 섹션으로 구분되지 않음

#### 실전 활용 패턴

##### 1. 메서드 체이닝
```java

public class ProductBuilder {
    private String name;
    private int price;

    public static ProductBuilder start() {
        return new ProductBuilder();
    }

    public ProductBuilder name(String name) {
        this.name = name;
        return this;
    }

    public ProductBuilder price(int price) {
        this.price = price;
        return this;
    }

    public Product build() {
        return new Product(name, price);
    }
}

// 사용 예시
Product product = ProductBuilder.start()
                               .name("Laptop")
                               .price(1500)
                               .build();
```

##### 2. 서비스 제공자 프레임워크
```java

// 서비스 인터페이스
public interface PaymentService {
    void pay(int amount);
}

// 서비스 제공자 등록 API
public class Services {
    private static final Map<String, PaymentService> providers = new HashMap<>();
    
    public static void registerProvider(String name, PaymentService provider) {
        providers.put(name, provider);
    }
    
    public static PaymentService getProvider(String name) {
        return providers.get(name);
    }
}

// 사용 예시
Services.registerProvider("credit", new CreditCardService());
PaymentService service = Services.getProvider("credit");
```


#### 핵심 정리

| 구분 | 생성자 | 정적 팩토리 메서드 |
|------|--------|-------------------|
| 유연성 | 낮음 | 높음 (이름, 캐싱, 타입 변환 등) |
| 성능 | 매번 새 객체 | 상황에 따라 재사용 가능 |
| 확장성 | 제한적 | 높음 (하위 타입 반환) |

### 6P

#### 1. 이름을 가진 생성자로 가독성 높이기
생성자만으로는 객체 생성의 의도를 명확히 표현하기 어렵습니다. 정적 팩토리 메서드를 사용하면 이런 문제를 해결할 수 있습니다.

```java
// 기존 생성자 방식
public class Order {
    private String product;
    private int quantity;
    
    public Order(String product, int quantity) { ... }
}

// 정적 팩토리 메서드 방식
public class Order {
    private String product;
    private int quantity;
    
    public static Order createBulkOrder(String product) {
        return new Order(product, 100);  // 대량 주문은 기본 100개
    }
    
    public static Order createSampleOrder(String product) {
        return new Order(product, 1);    // 샘플 주문은 1개
    }
}
```

이렇게 하면 코드만 봐도 의도를 쉽게 파악할 수 있습니다:
```java
Order bulk = Order.createBulkOrder("노트북");
Order sample = Order.createSampleOrder("마우스");
```

#### 2. 인스턴스 재사용으로 성능 개선하기
자주 사용되는 인스턴스는 매번 새로 생성하지 않고 캐시해서 재사용하면 성능을 높일 수 있습니다.

```java
public class License {
    private static final Map<String, License> CACHE = new HashMap<>();
    private String key;
    
    private License(String key) { this.key = key; }
    
    public static License of(String key) {
        return CACHE.computeIfAbsent(key, License::new);
    }
}
```

이렇게 하면 같은 키로 요청할 때 항상 동일한 인스턴스를 반환합니다:
```java
License a = License.of("A-123");  // 새로운 인스턴스 생성
License b = License.of("A-123");  // 기존 인스턴스 재사용
// a와 b는 같은 객체를 참조 (a == b는 true)
```

#### 3. 하위 타입 반환으로 유연성 확보하기
인터페이스를 통해 구현 세부사항을 숨기고, 클라이언트 코드를 더 유연하게 만들 수 있습니다.

```java
public interface Payment {
    void process();
    
    // 정적 팩토리 메서드로 구현체를 감춤
    static Payment getCreditPayment() {
        return new CreditCardPayment();
    }
    
    // 불변 내부 클래스로 구현
    private static class CreditCardPayment implements Payment {
        @Override
        public void process() {
            System.out.println("신용카드 결제 처리");
        }
    }
}
```

클라이언트는 구현 세부사항을 몰라도 됩니다:
```java
Payment payment = Payment.getCreditCardPayment();
```

> Java의 컬렉션 프레임워크(예: `List.of()`, `Collections.synchronizedList()`)에서 이런 패턴을 자주 볼 수 있습니다. 이 방식을 활용하면 구현 세부사항은 숨기고, 최적화된 객체를 제공할 수 있습니다.

### 7-8 P

#### 1. 인터페이스의 정적 메서드 (Java 8+)
Java 8부터는 인터페이스에 직접 정적 메서드를 정의할 수 있어서 별도의 동반 클래스가 필요 없어졌습니다.

```java
public interface Payment {
    void process();
    
    // 정적 팩토리 메서드로 구현체를 감춤
    static Payment getCreditPayment() {
        return new CreditCardPayment();
    }
    
    // 불변 내부 클래스로 구현
    private static class CreditCardPayment implements Payment {
        @Override
        public void process() {
            System.out.println("신용카드 결제 처리");
        }
    }
}
```

#### 2. EnumSet의 동적 구현체 반환
입력 매개변수에 따라 서로 다른 구현체를 반환할 수 있는 좋은 예시입니다.

```java
public class Main {
    public static void main(String[] args) {
        // 원소 개수에 따라 다른 구현체가 자동으로 선택됨
        EnumSet<Season> set = EnumSet.allOf(Season.class);
        // 64개 이하: RegularEnumSet
        // 65개 이상: JumboEnumSet
    }
}

enum Season { SPRING, SUMMER, FALL, WINTER }
```

#### 3. JDBC와 같은 서비스 제공자 프레임워크
정적 팩토리 메서드가 서비스 접근 API의 핵심 역할을 합니다.

```java
// 서비스 인터페이스
public interface Connection {
    void connect();
}

// 서비스 제공자 등록 및 접근 API
public class DriverManager {
    private static final Map<String, Driver> drivers = new HashMap<>();
    
    public static void registerDriver(String name, Driver driver) {
        drivers.put(name, driver);
    }
    
    public static Connection getConnection(String name) {
        return drivers.get(name).getConnection();
    }
}
```

#### 4. 불변 클래스와 상속 방지
public 생성자를 제공하지 않음으로써 상속을 방지하고 불변성을 보장할 수 있습니다.

```java
public final class ImmutablePoint {
    private final int x, y;
    
    // 정적 팩토리 메서드로 객체 생성
    public static ImmutablePoint of(int x, int y) {
        return new ImmutablePoint(x, y);
    }
    
    // private 생성자로 상속 차단
    private ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

#### 5. 동반 클래스 Collections 예시
Java 8 이전에는 동반 클래스 패턴이 널리 사용되었습니다.

```java
public class Collections {
    // 다양한 정적 팩토리 메서드 제공
    public static <T> List<T> unmodifiableList(List<? extends T> list) {
        return (list instanceof RandomAccess ?
                new UnmodifiableRandomAccessList<>(list) :
                new UnmodifiableList<>(list));
    }
    
    // 패키지-private 불변 내부 클래스
    static class UnmodifiableList<E> extends AbstractList<E> {
        // 구현 생략
    }
}
```

### 패턴별 관련 항목 정리

| 패턴 | 관련 Item | 핵심 개념 |
|------|-----------|-----------|
| 인터페이스 정적 메서드 | Item 1, 4, 17 | 캡슐화, 구현체 은닉 |
| EnumSet | Item 1, 36 | 동적 구현체 선택 |
| JDBC | Item 1, 5, 59 | 서비스 제공자 프레임워크 |
| 불변 클래스 | Item 1, 17, 18 | 불변성, 상속 제한 |
| Collections | Item 1, 4 | 동반 클래스 패턴 |

## ITEM 2: Consider a builder when faced with many constructor parameters

### P 9 - 13

#### 명명 규칙을 준수하라 

정적 팩토리 메서드의 이름은 관례적으로 다음과 같은 패턴을 따릅니다:

##### 1. from() - 단일 매개변수의 타입 변환
한 개의 매개변수를 받아서 해당 타입의 인스턴스를 반환합니다.

```java
public class Date {
    public static Date from(Instant instant) {
        return new Date(instant.toEpochMilli());
    }
}

// 사용 예시
Date date = Date.from(instant);
```

##### 2. of() - 여러 매개변수의 집계
여러 매개변수를 받아 적합한 타입의 인스턴스를 반환합니다.

```java
public class MyEnumSet<E extends Enum<E>> {
    public static <E extends Enum<E>> MyEnumSet<E> of(E e1, E e2, E e3) {
        return new MyEnumSet<>(Set.of(e1, e2, e3));
    }
}

// 사용 예시
MyEnumSet<Card> cards = MyEnumSet.of(JACK, QUEEN, KING);
```

##### 3. valueOf() - 상세한 타입 변환
from과 of의 더 자세한 버전입니다.

```java
public class BigInteger {
    public static BigInteger valueOf(long val) {
        if (val >= 0 && val < MAX_PRECOMPUTED)
            return precomputed[(int) val];  // 캐시된 값 반환
        return new BigInteger(val);
    }
}

// 사용 예시
BigInteger bi = BigInteger.valueOf(Integer.MAX_VALUE);
```

##### 4. getInstance() - 인스턴스 반환
매개변수로 명시한 인스턴스를 반환하지만, 같은 값이라도 다른 인스턴스일 수 있습니다.

```java
public class DatabaseConnection {
    private static DatabaseConnection INSTANCE;
    
    public static DatabaseConnection getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new DatabaseConnection();
        }
        return INSTANCE;
    }
}

// 사용 예시
DatabaseConnection conn = DatabaseConnection.getInstance();
```

##### 5. newInstance() - 새로운 인스턴스 생성
매번 새로운 인스턴스를 생성해서 반환합니다.

```java
public class Array {
    public static Object newInstance(Class<?> componentType, int length) {
        return java.lang.reflect.Array.newInstance(componentType, length);
    }
}

// 사용 예시
String[] arr = (String[]) Array.newInstance(String.class, 10);
```

##### 6. getType() - 다른 클래스의 팩토리 메서드
다른 클래스에서 Type의 인스턴스를 반환합니다.

```java
public final class Files {
    public static FileStore getFileStore(Path path) throws IOException {
        return path.getFileSystem().getFileStores().iterator().next();
    }
}

// 사용 예시
FileStore store = Files.getFileStore(Paths.get("/"));
```

##### 7. type() - 간결한 팩토리 메서드
getType이나 newType의 간단한 버전입니다.

```java
public class Collections {
    public static <T> ArrayList<T> list(Enumeration<T> e) {
        ArrayList<T> list = new ArrayList<>();
        while (e.hasMoreElements()) {
            list.add(e.nextElement());
        }
        return list;
    }
}

// 사용 예시
List<String> list = Collections.list(oldVector.elements());
```

#### 명명 규칙 요약표

| 메서드명 | 용도 | 예시 | 특징 |
|----------|------|------|-------|
| from | 타입 변환 | Date.from(instant) | 단일 매개변수 |
| of | 여러 매개변수 집계 | EnumSet.of(JACK, QUEEN) | 다중 매개변수 |
| valueOf | 상세한 변환 | BigInteger.valueOf(long) | 자세한 버전 |
| getInstance | 인스턴스 반환 | StackWalker.getInstance() | 동일성 미보장 |
| newInstance | 새 인스턴스 생성 | Array.newInstance() | 매번 새로 생성 |
| getType | 다른 클래스 팩토리 | Files.getFileStore() | 다른 클래스에서 반환 |
| type | 간결한 대안 | Collections.list() | 간단한 버전 |

#### 점층적 생성자 패턴(Telescoping Constructor)

```java
// 점층적 생성자 패턴 - 매개변수 증가시 확장 불가!
public class NutritionFacts {
    private final int servingSize;  // 필수
    private final int servings;     // 필수
    private final int calories;     // 선택
    private final int fat;          // 선택
    private final int sodium;       // 선택
    private final int carbohydrate; // 선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    // ... 계속되는 생성자들 (총 6개)
    
    public NutritionFacts(int sSize, int srvgs, int cal, int f, int sod, int carb) {
        this.servingSize = sSize;
        this.servings = srvgs;
        this.calories = cal;
        this.fat = f;
        this.sodium = sod;
        this.carbohydrate = carb;
    }
}

// 사용 예시: 원하지 않는 매개변수도 0으로 전달해야 함
NutritionFacts cola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

##### 문제점

- 6개 매개변수만으로도 가독성 저하

- 타입이 같은 매개변수 순서 오류 시 컴파일러가 감지 불가 (Item 51 위반)

- 매개변수 추가 시 모든 생성자 수정 필요

#### JavaBeans pattern

```java
// 자바빈즈 패턴 - 일관성 파괴 & 불변성 불가
public class NutritionFacts {
    private int servingSize = -1;  // 필수 (기본값 없음)
    private int servings = -1;     // 필수
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {}  // 필수 값 설정 강제 불가

    // Setter 메서드들
    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val)    { servings = val; }
    public void setCalories(int val)    { calories = val; }
    public void setFat(int val)         { fat = val; }
    public void setSodium(int val)      { sodium = val; }
    public void setCarbohydrate(int val){ carbohydrate = val; }
}

// 사용 예시
NutritionFacts cola = new NutritionFacts();
cola.setServingSize(240);
cola.setServings(8);
cola.setCalories(100);
cola.setSodium(35);
cola.setCarbohydrate(27);
// fat 필드 설정 누락 → 기본값 0 사용 (의도치 않은 동작)
```

##### 문제점

- 객체 생성이 여러 단계에 걸쳐 진행 → 일관성 무너짐

- setter 메서드로 인해 불변 객체 생성 불가 (Item 17 위반)

- 스레드 안전성 확보를 위해 추가 작업 필요 (Item 83 관련)

#### Builder pattern

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 (기본값 초기화)
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}

// 사용 예시
NutritionFacts cola = new NutritionFacts.Builder(240, 8)
    .calories(100)
    .sodium(35)
    .carbohydrate(27)
    .build();  // fat은 기본값 0 유지
```
#### 패턴 비교

| 패턴 | 장점 | 단점 |
|----------|------|------|-------|
| Telescoping | 불변 객체 보장 | 확장성 낮음, 가독성 나쁨 |
| JavaBeans | 유연한 설정 | 일관성 문제, 불변성 불가 |
| Builder | 유연성 + 불변성 + 가독성 | 코드 양 증가 |

결론적으로 Builder Pattern을 쓰는 게 제일 낫다!

### P 14-16

#### Builder pattern
```java
// 계층적 빌더 패턴
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }
        
        abstract Pizza build();
        protected abstract T self(); // 하위 클래스에서 "this" 반환
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // Item 50 참조
    }
}
```

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;
        
        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }
        
        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }
        
        @Override
        protected Builder self() {
            return this;
        }
    }
    
    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}

public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false;
        
        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }
        
        @Override
        public Calzone build() {
            return new Calzone(this);
        }
        
        @Override
        protected Builder self() {
            return this;
        }
    }
    
    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

#### 계층적 빌더 패턴의 고급 기능

##### 1. Recursive Generic Type
```java
abstract static class Builder<T extends Builder<T>>
```

- 목적: 모든 하위 빌더가 `self()`를 통해 자신의 타입을 반환하도록 강제
- 예시:
  - `NyPizza.Builder.addTopping()` → `NyPizza.Builder` 반환
  - `Calzone.Builder.sauceInside()` → `Calzone.Builder` 반환

##### 2. Covariant Return Type
```java
@Override
public NyPizza build() {  // 반환 타입을 구체화
    return new NyPizza(this);
}
```

- 효과: 클라이언트 코드에서 형변환 불필요
```java
NyPizza pizza = new NyPizza.Builder(Size.LARGE).build(); // (O)
// Pizza pizza = ... → (X) 불필요한 업캐스팅 방지
```

##### 3. Simulated Self-Type
```java
protected abstract T self();
```

- 해결 문제: 자바의 `this` 한계 (항상 현재 클래스 타입)
- 구현 예시:
```java
// NyPizza.Builder
@Override
protected Builder self() { return this; }  // 실제 반환 타입 지정
```

#### 빌더 패턴의 고급 기능

##### 1. 다중 가변인수 처리
```java
public Builder addToppings(Topping... toppings) {
    Collections.addAll(this.toppings, toppings);
    return self();
}
```

##### 2. 동적 기본값 설정
```java
public Builder setDefaultSauce() {
    if (sauce == null) sauce = new TomatoSauce();
    return self();
}
```

#### 장단점 분석

##### 장점
- 확장성: 새로운 피자 타입 추가 시 기존 코드 변경 불필요
- Type Safety: `NyPizza.Builder`는 뉴욕 피자 전용 설정만 허용
- Immutability: 모든 필드 `final` 처리 가능 (스레드 안전)

##### 단점
- 복잡성 증가: 작은 클래스에선 과도한 boilerplate 코드
- 성능 오버헤드: 빌더 생성 비용 (고성능 시스템 고려 시)

#### 적용 시 고려사항

| 상황 | 권장 접근법 |
|------|------------|
| 필드 4개 이상 | 빌더 패턴 필수 적용 |
| 동일 타입 연속 매개변수 | 빌더로 오류 예방 |
| 향후 확장 예상 | 초기부터 빌더 구현 |
| 간단한 DTO | 생성자/정적 팩토리 |

## ITEM 3: Enforce the singleton property with a private constructor or an enum type

### P 17 - 18

#### 싱글톤 구현 방법

##### 1. Public Final Field 방식
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    
    private Elvis() { 
        // Reflection 공격 방지
        if (INSTANCE != null) {
            throw new IllegalStateException("이미 인스턴스가 존재합니다");
        }
        // 초기화 코드
    }
    
    public void leaveTheBuilding() {
        System.out.println("엘비스가 건물을 떠났습니다!");
    }
}
```

##### 동작 원리

```java
Elvis elvis = Elvis.INSTANCE; // 즉시 사용 가능

// 컴파일 오류: private 생성자
new Elvis(); 

// Reflection 공격 시도 실패
Constructor<Elvis> constructor = Elvis.class.getDeclaredConstructor();
constructor.setAccessible(true); // 실패! 생성자에서 예외 발생
```

##### 2. Static Factory Method 방식
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    
    private Elvis() { /* 초기화 코드 */ }
    
    public static Elvis getInstance() {
        return INSTANCE;
    }
    
    public void dance() {
        System.out.println("엘비스가 댄스를 춥니다!");
    }
}
```

##### 동작 확인
```java
Elvis elvis1 = Elvis.getInstance();
Elvis elvis2 = Elvis.getInstance();
System.out.println(elvis1 == elvis2); // true (동일 객체)
```

#### 두 방식 비교

| 기준 | Public Field | Static Factory |
|------|--------------|----------------|
| 간결성 | ⭐️⭐️⭐️⭐️ | ⭐️⭐️⭐️ |
| 유연성 | 확장 불가 | 구현 변경 가능 |
| Serialization | 추가 작업 필요 | readResolve() 구현 필요 |

#### 주의사항

##### 1. Reflection 공격 방어
```java
private Elvis() {
    if (INSTANCE != null) {
        throw new IllegalStateException();
    }
}
```

##### 2. Serialization 문제 해결
```java
// 싱글톤 속성 유지를 위한 readResolve 메서드
private Object readResolve() {
    // 진짜 엘비스 반환, 가짜는 가비지 컬렉터에 맡김
    return INSTANCE;
}
```

##### 3. Testing 용이성
```java
public interface Singer {
    void sing();
}

public class MockElvis implements Singer {
    @Override public void sing() { /* 모의 구현 */ }
}
// 테스트 시 Mock 주입 불가 (Elvis가 인터페이스 구현 안 할 경우)
```

#### 권장 방식: Enum Singleton
```java
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() { 
        System.out.println("Bye!"); 
    }
}
```

```java
public enum ElvisEnum {
    INSTANCE;
    
    public void perform() {
        System.out.println("Viva Las Vegas!");
    }
    
    // 사용 예시
    public static void main(String[] args) {
        ElvisEnum.INSTANCE.perform();
    }
}
```

#### 구현 방식 선택 기준

| 상황 | 권장 방식 |
|------|-----------|
| 단순함 추구 | Public Final Field |
| 유연성 필요 | Static Factory Method |
| 최선의 선택 | Enum Singleton |

> Enum Singleton 방식은 Reflection과 Serialization 문제를 근본적으로 해결합니다.

## TIEM 4: Enforce noninstantiability with a private constructor

### P 19

#### 유틸리티 클래스란?

유틸리티 클래스는 정적 메서드와 정적 필드로만 구성된 클래스입니다. 이 클래스는 인스턴스화될 필요가 없으며, 일반적으로 여러 관련 기능을 그룹화하는 데 사용됩니다.

#### 문제점

유틸리티 클래스는 종종 의도치 않게 인스턴스화될 수 있습니다. 예를 들어, 기본 생성자가 제공되면 사용자가 생성자를 호출하여 인스턴스를 생성할 수 있습니다.

#### 유틸리티 클래스가 필요한 상황

##### 1. 기본 타입이나 배열 관련 작업을 모아둘 때

```java
// 수학 연산 유틸리티
public class MathOperations {
    public static int add(int a, int b) { return a + b; }
    public static int subtract(int a, int b) { return a - b; }
    public static int multiply(int a, int b) { return a * b; }
}

// 배열 유틸리티
public class ArrayUtils {
    public static void reverse(int[] array) {
        for (int i = 0; i < array.length / 2; i++) {
            int temp = array[i];
            array[i] = array[array.length - 1 - i];
            array[array.length - 1 - i] = temp;
        }
    }
}
```
##### 2. 정적 팩토리 메서드를 모아둘 때

```java
public interface Animal {
    void makeSound();
}

public class Animals {
    public static Animal createDog() {
        return new Dog();
    }
    
    public static Animal createCat() {
        return new Cat();
    }
}
```

#### 잘못된 유틸리티 클래스 구현

##### 1. 생성자를 명시하지 않은 경우

```java
public class StringUtils {
    // 기본 생성자가 자동으로 생성됨 (public)
    
    public static String reverse(String str) {
        return new StringBuilder(str).reverse().toString();
    }
}

// 문제점: 인스턴스화 가능
StringUtils utils = new StringUtils(); // 컴파일 성공!
```

##### 2. abstract 클래스로 시도한 경우
```java
public abstract class DatabaseUtils {
    public static void executeQuery(String sql) {
        // 쿼리 실행 로직
    }
}

// 문제점: 상속으로 인스턴스화 가능
public class MyDatabaseUtils extends DatabaseUtils {
    // 인스턴스화 가능!
}
```

#### 올바른 유틸리티 클래스 구현

##### 1. private 생성자로 인스턴스화 방지

```java

public class FileUtils {
    // 인스턴스화 방지
    private FileUtils() {
        throw new AssertionError("이 클래스는 인스턴스화할 수 없습니다!");
    }
    
    public static String readFile(String path) {
        // 파일 읽기 로직
        return "file contents";
    }
    
    public static void writeFile(String path, String content) {
        // 파일 쓰기 로직
    }
}

// 사용
FileUtils.readFile("test.txt");  // OK
new FileUtils();  // 컴파일 에러!
```

##### 2. 상속 방지 효과
```java
public class MyFileUtils extends FileUtils {  // 컴파일 에러!
    // private 생성자로 인해 상속 불가능
}
```

#### 사용 예시

```java
// 날짜 관련 유틸리티
public class DateUtils {
    private DateUtils() {
        throw new AssertionError();
    }
    
    public static String formatDate(Date date) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        return sdf.format(date);
    }
    
    public static Date parseDate(String dateStr) throws ParseException {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        return sdf.parse(dateStr);
    }
}

// 사용
String formattedDate = DateUtils.formatDate(new Date());  // 정상 동작
Date parsedDate = DateUtils.parseDate("2024-03-19");     // 정상 동작
```

#### 장단점

| 방법 | 장점 | 단점 |
|------|------|------|
| private 생성자 | 100% 인스턴스화 방지, 상속 차단 | 주석 필요 |
| 추상 클래스 | 의미 전달 용도로만 사용 | 실제 방지 효과 없음 |
| 인터페이스 정적 메서드 | 자연스러운 그룹화 | 인터페이스 수정 권한 필요 |

## Item 5: Prefer dependency injection to hardwiring resources

클래스가 하나 이상의 자원에 의존할 때, 정적 유틸리티 클래스나 싱글톤을 사용하는 것은 큰 실수입니다.
(예: 맞춤법 검사기가 사전에 의존하는 경우)

### 잘못된 패턴

#### 1. 정적 유틸리티 클래스
```java
public class SpellChecker {
    private static final Lexicon dictionary = new KoreanDictionary(); // 하드코딩된 의존성
    
    private SpellChecker() {} // 인스턴스화 방지
    
    public static boolean isValid(String word) { 
        return dictionary.isValid(word);
    }
}
```

**문제점:**
- 다른 사전(영어, 테스트용)으로 교체 불가
- 동시성 제어 어려움
- 테스트 시 Mock 주입 불가

#### 2. 싱글톤
```java
public class SpellChecker {
    private final Lexicon dictionary = new KoreanDictionary();
    
    private SpellChecker() {}
    public static final SpellChecker INSTANCE = new SpellChecker();
    
    public boolean isValid(String word) {
        return dictionary.isValid(word);
    }
}
```

**문제점:**
- 모든 클라이언트가 동일한 사전 강제
- 런타임 환경별 설정 변경 불가

### 올바른 해결책

#### 1. 의존성 주입(DI)
```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    // 생성자를 통해 의존성 주입
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) {
        return dictionary.isValid(word);
    }
}

// 사용 예시
Lexicon testDictionary = new TestDictionary(); 
SpellChecker checker = new SpellChecker(testDictionary); // 테스트용 사전 주입
```

#### 2. 팩토리 주입 패턴 (Java 8+ Supplier 활용)
```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    // 팩토리 주입 (동적 생성 가능)
    public SpellChecker(Supplier<? extends Lexicon> factory) {
        this.dictionary = factory.get();
    }
}

// 사용 예시
SpellChecker englishChecker = new SpellChecker(EnglishDictionary::new);
SpellChecker scienceChecker = new SpellChecker(ScienceDictionary::new);
```

#### 3. 의존성 주입 프레임워크 활용
```java
// Spring 예시
@Configuration
public class AppConfig {
    @Bean
    public Lexicon koreanDictionary() {
        return new KoreanDictionary();
    }
}

@Service
public class SpellChecker {
    private final Lexicon dictionary;
    
    @Autowired // 스프링이 의존성 자동 주입
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = dictionary;
    }
}
```

### 전통적 방식 vs 의존성 주입 비교

| 기준 | 정적 유틸리티/싱글톤 | 의존성 주입 |
|------|---------------------|------------|
| 유연성 | 낮음 (단일 자원) | 높음 (런타임 결정) |
| 테스트 용이성 | 어려움 (Mock 불가) | 용이 (주입 가능) |
| 병렬 처리 | 위험 (공유 상태) | 안전 (독립 인스턴스) |
| 확장성 | 제한적 | 무제한 (팩토리/상속 조합) |

### 핵심 정리
1. 클래스는 자신이 사용하는 자원을 직접 생성하지 말라
2. 외부에서 필요한 자원을 주입받아 사용하라
3. 생성자/정적 팩토리/빌더 모두 DI의 유효한 수단이다

## Item 6: Avoid creating unnecessary objects

불필요한 객체 생성을 피하는 것은 성능을 개선하고 메모리 사용량을 줄이는 데 도움이 됩니다. 객체를 재사용하는 것이 더 빠르고 효율적일 수 있으며, 특히 불변 객체(immutable objects)는 언제든지 안전하게 재사용할 수 있습니다.

### 1. 문자열 객체 생성

```java
// 잘못된 예 - 새로운 String 인스턴스를 매번 생성
String s1 = new String("bikini");  // DON'T DO THIS!

// 올바른 예 - String 리터럴 사용
String s2 = "bikini";  // 문자열 풀에서 재사용
```

### 2. 정적 팩토리 메서드 활용

```java
// 잘못된 예 - 생성자 사용
Boolean aBoolean = new Boolean("true");  // Deprecated in Java 9

// 올바른 예 - 정적 팩토리 메서드 사용
Boolean bBoolean = Boolean.valueOf("true");  // 캐시된 인스턴스 재사용
```

### 3. 비싼 객체 캐싱

#### 3.1 정규표현식 패턴 재사용
```java
// 잘못된 예 - Pattern 인스턴스를 매번 생성
public class RomanNumerals {
    static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})...");  // Pattern 객체를 매번 생성
    }
}

// 올바른 예 - Pattern 인스턴스를 캐싱하여 재사용
public class RomanNumerals {
    private static final Pattern ROMAN = 
        Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();  // 컴파일된 Pattern 재사용
    }
}
```

#### 3.2 키파생 메서드 캐싱
```java
// 비싼 객체를 캐시하는 예시
public class ExpensiveObjectCache {
    private static final Map<String, ExpensiveObject> cache = new ConcurrentHashMap<>();
    
    public static ExpensiveObject getInstance(String key) {
        return cache.computeIfAbsent(key, ExpensiveObject::create);
    }
}
```

### 4. 어댑터 패턴의 효율적 사용

```java
// Map 인터페이스의 keySet 어댑터 예시
public class MapAdapter {
    private final Map<String, Integer> map = new HashMap<>();
    private Set<String> keySet;  // keySet 뷰를 캐시

    public Set<String> getKeySet() {
        Set<String> result = keySet;
        if (result == null) {
            keySet = result = map.keySet();  // 동일한 Set 인스턴스 재사용
        }
        return result;
    }
}
```

### 5. 오토박싱 주의사항

```java
// 잘못된 예 - 불필요한 오토박싱
private static long sumSlow() {
    Long sum = 0L;  // 박싱된 기본 타입 사용
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;  // 매번 새로운 Long 인스턴스 생성
    }
    return sum;
}

// 올바른 예 - 기본 타입 사용
private static long sum() {
    long sum = 0L;  // 기본 타입 사용
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;  // 오토박싱 없음
    }
    return sum;
}
```

### 6. 객체 풀링 고려사항

```java
// 데이터베이스 연결 풀 예시 - 무거운 객체에만 적용
public class DatabaseConnectionPool {
    private static final int MAX_POOL_SIZE = 10;
    private final BlockingQueue<Connection> pool;
    
    public DatabaseConnectionPool() {
        pool = new ArrayBlockingQueue<>(MAX_POOL_SIZE);
        for (int i = 0; i < MAX_POOL_SIZE; i++) {
            pool.offer(createConnection());
        }
    }
    
    public Connection getConnection() throws InterruptedException {
        return pool.take();  // 풀에서 연결 획득
    }
    
    public void releaseConnection(Connection conn) {
        pool.offer(conn);  // 풀에 연결 반환
    }
}
```

### 성능 비교 예시

```java
public class ObjectCreationBenchmark {
    public static void main(String[] args) {
        // String 생성 성능 비교
        long start = System.nanoTime();
        for (int i = 0; i < 1000000; i++) {
            String s = new String("test");  // 비효율적
        }
        long end = System.nanoTime();
        System.out.println("New String time: " + (end - start));

        start = System.nanoTime();
        for (int i = 0; i < 1000000; i++) {
            String s = "test";  // 효율적
        }
        end = System.nanoTime();
        System.out.println("String literal time: " + (end - start));
    }
}
```

### 핵심 정리

1. **재사용이 핵심**: 불변 객체는 항상 재사용할 수 있습니다.
2. **정적 팩토리 메서드**: 객체 재사용을 보장하는 좋은 방법입니다.
3. **비싼 객체는 캐싱**: 생성 비용이 비싼 객체는 캐싱하여 재사용합니다.
4. **기본 타입 선호**: 불필요한 오토박싱을 피하기 위해 기본 타입을 사용합니다.
5. **객체 풀은 신중히**: 매우 무거운 객체가 아니라면 객체 풀은 피합니다.

| 상황 | 권장 방식 | 피해야 할 방식 |
|------|-----------|---------------|
| 문자열 생성 | String 리터럴 | new String() |
| 불변 객체 | 정적 팩토리 메서드 | 생성자 |
| 비싼 객체 | 캐싱 후 재사용 | 매번 새로 생성 |
| 기본 타입 | primitive 타입 | 박싱된 기본 타입 |
| 무거운 객체 | 객체 풀링 고려 | 매번 새로 생성 |

## Item 7: Eliminate obsolete object references

가비지 컬렉션(GC) 언어인 Java에서도 메모리 관리는 중요합니다. 의도치 않은 객체 보유(unintentional object retention)는 메모리 누수로 이어질 수 있으며, 성능 저하 또는 OutOfMemoryError를 초래할 수 있습니다.

### 메모리 누수의 주요 원인과 해결책

#### 1. 자체 메모리 관리 클래스의 메모리 누수

Stack 구현의 메모리 누수 예시:

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    // 메모리 누수가 있는 버전
    public Object pop() {
        if (size == 0) throw new EmptyStackException();
        return elements[--size]; // 참조 해제를 하지 않음
    }

    // 해결책: 다 쓴 참조 해제
    public Object popFixed() {
        if (size == 0) throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

#### 2. 캐시 관련 메모리 누수

```java
// 캐시 구현의 메모리 누수 해결책
public class Cache<K, V> {
    // WeakHashMap을 사용하여 참조가 없는 항목 자동 제거
    private final Map<K, V> cache = new WeakHashMap<>();
    
    // 또는 만료 시간 설정
    private final Map<K, CacheEntry<V>> timedCache = new HashMap<>();
    
    private static class CacheEntry<V> {
        V value;
        long expiryTime;
    }
    
    public V get(K key) {
        cleanExpiredEntries();
        return timedCache.get(key).value;
    }
    
    private void cleanExpiredEntries() {
        long now = System.currentTimeMillis();
        timedCache.entrySet().removeIf(e -> e.getValue().expiryTime < now);
    }
}
```

#### 3. 리스너/콜백 관련 메모리 누수

```java
public class EventManager {
    // WeakHashMap을 사용하여 콜백 참조 관리
    private final Map<EventListener, Object> listeners = new WeakHashMap<>();
    
    public void addListener(EventListener listener) {
        listeners.put(listener, new Object());
    }
    
    public void removeListener(EventListener listener) {
        listeners.remove(listener);
    }
}
```

### 메모리 누수 방지를 위한 가이드라인

#### 1. 명시적 참조 해제가 필요한 경우
- 자체적으로 메모리를 관리하는 클래스
- 캐시
- 리스너/콜백

#### 2. 참조 해제 방법
- null 처리
- WeakHashMap 사용
- 백그라운드 스레드를 이용한 주기적 청소

#### 3. 메모리 누수 디버깅

```java
// 메모리 사용량 모니터링 예시
public class MemoryMonitor {
    public static void printMemoryStats() {
        Runtime rt = Runtime.getRuntime();
        System.out.println("Used Memory: " + 
            (rt.totalMemory() - rt.freeMemory()) / 1024 + " KB");
    }
}
```

### 핵심 정리

| 상황 | 해결 방법 | 주의사항 |
|------|-----------|----------|
| 자체 메모리 관리 | 명시적 null 처리 | 일반적인 상황에서는 불필요 |
| 캐시 | WeakHashMap 또는 TTL | 캐시 크기 제한 고려 |
| 리스너/콜백 | 약한 참조 사용 | 등록/해제 쌍으로 관리 |

> "메모리 누수는 눈에 띄지 않게 서서히 시스템을 죽입니다. 객체 참조의 생명주기를 항상 의식해야합니다"

## ITEM 8: Avoid finalizers and cleaners

파이널라이저(finalizers)와 클리너(cleaners)는 예측 불가능하고 위험하며 일반적으로 불필요합니다. 이들의 사용은 불안정한 동작, 성능 저하, 이식성 문제를 초래할 수 있습니다. Java 9부터 파이널라이저는 공식적으로 deprecated되었지만 여전히 일부 라이브러리에서 사용 중입니다. 클리너가 대체 수단으로 도입되었지만 여전히 예측 불가능하고 느리며 권장되지 않습니다.

C++ 개발자들은 파이널라이저/클리너를 C++의 소멸자(destructor)와 동일시하지 말아야 합니다. C++에서는 소멸자가 객체 리소스 회수의 표준 방법이지만, Java에서는 가비지 컬렉터가 자동으로 관리합니다. Java에서 비메모리 리소스 회수는 try-with-resources 또는 try-finally 블록으로 처리해야 합니다(Item 9).

### 파이널라이저의 치명적 문제점

#### 1. 실행 시점 보장 없음
객체가 도달 불가능(unreachable) 상태가 된 후 실제 실행까지 시간 차이가 클 수 있습니다.

```java
public class Resource {
    private FileInputStream fis;
    
    public Resource(String path) throws FileNotFoundException {
        fis = new FileInputStream(path);
    }
    
    @Override
    protected void finalize() throws Throwable {
        fis.close(); // 파일 핸들 누수 위험
        super.finalize();
    }
}
```

#### 2. 성능 저하

```java
// 수만 개 객체 파이널라이즈 지연 사례
public class GraphicObject {
    private byte[] heavyResource = new byte[1024 * 1024];
    
    @Override
    protected void finalize() {
        // 장시간 실행 코드
    }
}
```

#### 3. 실행 보장 없음

```java
public class CriticalResource {
    private static Lock dbLock = ...;
    
    @Override
    protected void finalize() {
        dbLock.unlock(); // 실행 안 될 경우 분산 시스템 장애
    }
}
```

### 클리너(Cleaner)의 한계 (Java 9+)

```java
import java.lang.ref.Cleaner;

public class CleanerExample {
    private static final Cleaner cleaner = Cleaner.create();
    
    private static class Resource implements Runnable {
        private FileInputStream fis;
        
        Resource(String path) throws FileNotFoundException {
            fis = new FileInputStream(path);
        }
        
        @Override
        public void run() { // 클리닝 액션
            try {
                fis.close();
                System.out.println("리소스 클린!");
            } catch (IOException e) { /* 로깅 */ }
        }
    }
    
    public static void main(String[] args) {
        Resource res = new Resource("data.txt");
        cleaner.register(res, res::run); // 클리너 등록
        res = null; // 도달 불가능 상태
        // GC 시점에 클리닝 액션 실행 (보장 없음)
    }
}
```

### 올바른 리소스 관리 기법

```java
// try-with-resources (Java 7+)
public class SafeResource implements AutoCloseable {
    private final FileInputStream fis;
    
    public SafeResource(String path) throws FileNotFoundException {
        fis = new FileInputStream(path);
    }
    
    @Override
    public void close() {
        try {
            fis.close();
            System.out.println("리소스 즉시 해제!");
        } catch (IOException e) { /* 로깅 */ }
    }
    
    public static void main(String[] args) {
        try (SafeResource res = new SafeResource("data.txt")) {
            // 리소스 사용
        } // 자동 close() 호출 보장
    }
}
```

### 핵심 정리

#### 절대 사용하지 말 것
- finalize(): 100% 금지
- Cleaner: 네이티브 리소스 관리 등 극히 제한적 상황에서만

#### 대체 솔루션
- AutoCloseable + try-with-resources
- 명시적 close() 메서드 호출

#### 예외적 허용 사례
- 네이티브 피어(Native Peer) 리소스 관리
- 안전망 역할(주요 리소스 누수 방지)

> "파이널라이저와 클리너는 절대적인 최후의 수단입니다. 99.9%의 상황에서 try-with-resources가 정답입니다"

## Item 9: Prefer try-with-resources to try-finally

자바 라이브러리에는 `InputStream`, `OutputStream`, `java.sql.Connection`과 같이 직접 `close` 메서드를 호출해 수동으로 닫아야 하는 자원들이 많습니다. 자원 닫기는 클라이언트가 놓치기 쉬워 성능 문제로 이어지곤 합니다. 이런 자원들 중 상당수가 안전망으로 finalizer를 활용하지만, finalizer는 믿을 만하지 못합니다(아이템 8).

### try-finally의 문제점

역사적으로 `try-finally` 문은 예외 발생이나 return으로 인해 종료되는 상황에서도 자원이 제대로 닫힘을 보장하는 최선의 방법이었습니다.

```java
// try-finally - 더 이상 자원을 닫는 최선의 방법이 아니다!
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

여러 자원을 사용할 때는 코드가 더욱 복잡해집니다:

```java
// 여러 자원을 사용할 때 try-finally는 지저분하다!
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

### try-with-resources의 해결책

Java 7에서 도입된 try-with-resources는 이러한 문제들을 해결합니다. 이 구조를 사용하려면 해당 자원이 `AutoCloseable` 인터페이스를 구현해야 합니다.

```java
// try-with-resources - 자원 닫기 최적의 방식!
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

다중 자원 처리도 간단해집니다:

```java
// 다중 자원 처리 시 간결함의 극치
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

catch 절도 자유롭게 사용할 수 있습니다:

```java
// catch 결합 예시: 파일 열기 실패 시 기본값 반환
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

### try-with-resources의 장점

1. **간결성**: 코드가 짧고 명확해집니다
2. **예외 처리 개선**: 
   - 실제 발생한 예외가 명확히 표시됨
   - 추가 예외는 suppressed로 처리되어 `Throwable.getSuppressed()`로 확인 가능
3. **자원 관리 보장**: 자동으로 close() 호출

### 커스텀 리소스 예시

```java
public class CustomResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        System.out.println("리소스 정리 완료!");
    }
}

// 사용
try (CustomResource cr = new CustomResource()) {
    // 리소스 사용
} // 자동으로 close() 호출
```

### 비교 표: try-finally vs try-with-resources

| 특성 | try-finally | try-with-resources |
|------|-------------|-------------------|
| 가독성 | 중첩 코드로 복잡함 | 깔끔한 자원 선언 |
| 예외 처리 | 마지막 예외만 표시 | 모든 예외 보존 |
| 자원 누수 위험 | 높음 | 낮음 |
| 코드 길이 | 장황함 | 간결함 |

### 핵심 정리

- 꼭 회수해야 하는 자원을 다룰 때는 try-with-resources를 사용하라
- 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다
- try-finally로는 불가능했던 예외 처리가 가능해진다