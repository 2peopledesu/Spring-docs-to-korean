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