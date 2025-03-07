---
title: Variables and Primitive Data Types
date: 2025-03-07
weight: 2
---

### 1장 요약

1장에서는 컴파일러와 JVM을 소개했습니다. 우리는 첫 번째 자바 프로그램인 HelloWorld를 작성할 때 명령줄에서 이 둘을 사용하는 방법을 배웠습니다. 또한, 강력하고 친숙한 IDE인 IntelliJ를 소개하고, 여기에서도 HelloWorld를 실행했습니다.

모든 프로그래밍 언어는 변수를 요구하며, 내장된 기본 데이터 타입을 제공합니다. 이들은 가장 간단한 프로그램의 작동에도 필수적입니다. 이 장이 끝날 때까지 여러분은 자바의 기본 타입을 사용하여 변수를 선언할 수 있게 될 것입니다. 또한, 다양한 기본 데이터 타입 간의 차이점과 특정 상황에서 어떤 타입을 사용해야 하는지도 이해하게 될 것입니다.

이 장에서는 다음과 같은 주요 주제를 다룰 것입니다:

- 변수 이해 및 선언하기
- 자바의 기본 데이터 타입 탐색하기
- 기술 요구사항
이 장의 코드는 [GitHub](https://github.com/PacktPublishing/Learn-Java-with-Projects/tree/main/ch2)에서 확인할 수 있습니다.

### 변수 이해 및 선언하기

값을 나중에 사용하기 위해 저장하려면 변수가 필요합니다. 따라서 모든 프로그래밍 언어는 변수를 통해 이 기능을 제공합니다. 이 섹션에서는 변수가 무엇인지, 그리고 어떻게 선언하는지 배워보겠습니다. 코드에서 특정 변수를 사용할 수 있는 영역을 변수의 범위(scope)라고 합니다. 이는 매우 중요한 개념이며 4장에서 자세히 다룰 것입니다.

#### 변수란 무엇인가?
변수는 이름(식별자)과 타입을 가진 메모리의 위치입니다. 이는 이름이 지정된 우편함이나 우체국의 박스와 비슷합니다(그림 2.1 참조). 변수의 이름은 우리가 변수를 참조하고 다른 변수와 구별할 수 있도록 필요합니다.

변수의 타입은 저장할 수 있는 값의 종류를 명시합니다. 예를 들어, 이 변수가 4와 같은 정수를 저장할 것인지, 아니면 2.98과 같은 소수점을 저장할 것인지에 대한 질문이 그 변수의 타입을 결정합니다.

#### 변수 선언하기
예를 들어, 숫자 25를 변수에 저장하고 싶다고 가정해 봅시다. 이 숫자가 사람의 나이를 나타낸다고 가정하므로, 우리는 이를 참조하기 위해 age라는 식별자를 사용할 것입니다. 변수를 처음 도입하는 것을 “변수 선언(declaring)”이라고 합니다.

정수(양수 또는 음수)는 int라는 자바의 내장 기본 타입으로 제공됩니다. 기본 데이터 타입에 대해서는 다음 섹션에서 더 자세히 다룰 것입니다. 자바에서 변수를 선언할 때, 변수의 타입을 지정해야 합니다. 이는 자바가 강타입 언어(strongly typed language)로 알려져 있기 때문입니다. 즉, 변수를 선언할 때 즉시 변수의 타입을 지정해야 합니다.

변수를 선언하고 타입을 지정한 후 초기화해 봅시다:

```java
int age;
age = 25;
```

첫 번째 줄은 age를 int로 선언하고, 두 번째 줄은 25라는 값을 할당합니다. 각 줄 끝의 세미콜론(;)은 자바 문장이 끝나는 위치를 컴파일러에 알려주는 구분자입니다. = 기호는 할당 연산자로, 3장에서 다룰 예정입니다. 지금은 25가 age 변수에 “할당된다”는 것을 이해하세요.

#### 할당 연산자
자바에서 = 기호는 수학에서 사용하는 같음 기호(=)와 다릅니다. 자바는 같음을 나타내기 위해 == 기호를 사용하며, 이를 동등성(equivalence)이라고 부릅니다.

이전 두 줄의 코드를 한 줄로 작성할 수 있습니다:
```java
int age = 25;
```
### 변수 이름 짓기

식별자는 코딩하는 Java 구성 요소에 부여하는 이름입니다. 예를 들어, 식별자는 변수, 메서드, 클래스 등을 이름 짓는 데 필요합니다.

### 식별자

식별자는 문자, 숫자, 밑줄, 통화 기호로 구성됩니다. 식별자는 숫자로 시작할 수 없으며 공백(공백, 탭, 줄 바꿈)을 포함할 수 없습니다. 다음은 유효한 식별자의 예입니다: `a£€_23`, `_29`, `Z`, `thisIsAnExampleOfAVeryLongVariableName`, `€2_`, `$4`. 반면, 유효하지 않은 식별자의 예는 `9age`와 `abc def`입니다.

변수 이름을 신중하게 짓는 것이 중요합니다. 이는 코드의 가독성을 높여 버그를 줄이고 유지 관리를 쉽게 합니다. 카멜 케이스(camel case)는 이와 관련하여 매우 인기 있는 기법입니다. 변수 이름에 대한 카멜 케이스는 첫 번째 단어는 모두 소문자로 시작하고, 이후 각 단어의 첫 글자는 대문자로 시작하는 것을 의미합니다. 예를 들어:

```java
int ageOfCar;
int numberOfChildren;
```

이 코드 조각에서는 카멜 케이스를 따르는 두 개의 정수 변수가 있습니다.

### 변수 접근하기

변수의 값을 접근하려면 변수의 이름을 입력하면 됩니다. Java에서 변수의 이름을 입력하면, 컴파일러는 먼저 해당 이름의 변수가 존재하는지 확인합니다. 변수가 존재한다면, JVM은 런타임에 해당 변수의 값을 반환합니다. 따라서 다음 코드는 화면에 25를 출력합니다:

```java
int age = 25;
System.out.println(age);
```

첫 번째 줄은 `age` 변수를 선언하고 25로 초기화합니다. 두 번째 줄은 변수의 위치에 접근하여 그 값을 출력합니다.

### System.out.println()

`System.out.println()`은 괄호 안에 있는 내용을 화면에 출력합니다.

### 선언되지 않은 변수 접근하기

Java는 강타입 언어로 알려져 있습니다. 이는 변수를 선언할 때 즉시 변수의 타입을 지정해야 함을 의미합니다. 컴파일러가 변수를 발견했지만 그 타입을 알지 못하면 오류를 발생시킵니다. 예를 들어, 다음 코드 줄을 고려해 보십시오:

```java
age = 25;
```

다른 코드에서 `age`를 선언하지 않았다면, 컴파일러는 'age' 기호를 해결할 수 없다는 오류를 발생시킵니다. 이는 컴파일러가 `age`와 연관된 타입을 찾고 있지만 찾을 수 없기 때문입니다(타입을 지정하지 않았기 때문에).

조금 다른 예를 살펴보겠습니다. 이 예에서는 `age` 변수를 화면에 출력하려고 합니다:

```java
int length = 25;
System.out.println(age);
```

이 예에서는 `length`라는 변수를 선언했지만 `age`에 대한 선언이 없습니다. `System.out.println()`에서 `age` 변수를 접근하려고 할 때, 컴파일러는 `age`를 찾으러 갑니다. 컴파일러는 `age`를 찾을 수 없고 'age' 기호를 해결할 수 없다는 오류를 발생시킵니다. 이는 컴파일러가 찾을 수 없는 `age`라는 변수를 사용하려고 시도했음을 의미합니다. 이는 변수를 선언하지 않았기 때문입니다.

### 주석

주석은 코드에서 무슨 일이 일어나고 있는지를 설명하는 데 매우 유용합니다. 

- `//`는 단일 행 주석입니다. 컴파일러는 `//`를 만나면 해당 행의 나머지를 무시합니다.
- `/* some text */`는 다중 행 주석입니다. 여는 `/*`와 닫는 `*/` 사이의 모든 내용은 무시됩니다. 이 형식은 각 행의 시작에 `//`를 삽입하는 것을 방지합니다.

다음은 몇 가지 예입니다:

```java
int age; // 여기서부터 행의 나머지 부분은 무시됩니다
// 이 전체 행은 무시됩니다
/* 이
모든
행은
무시됩니다 */
```

Java는 강타입 언어로 모든 변수가 데이터 타입을 가져야 하므로, 이제 Java의 기본 데이터 타입에 대한 지원을 논의하겠습니다.

## Java의 기본 데이터 타입 이해하기

Java는 8개의 내장 데이터 타입을 제공합니다. 내장이라는 것은 이러한 데이터 타입이 언어와 함께 제공된다는 의미입니다. 이 기본 데이터 타입이 이 섹션의 주제입니다.

### Java의 기본 데이터 타입

모든 기본 데이터 타입은 소문자만 사용하여 이름이 지어집니다. 예를 들어, `int`와 `double`이 있습니다. 나중에 클래스, 레코드 및 인터페이스와 같은 사용자 정의 데이터 타입을 만들 때는 다른 명명 규칙을 따릅니다. 예를 들어, `Person` 또는 `Cat`이라는 이름의 클래스가 있을 수 있습니다. 이는 널리 채택된 코딩 규칙일 뿐이며, 컴파일러는 명명 규칙을 구분하지 않습니다. 그러나 기본 데이터 타입은 항상 소문자로만 되어 있기 때문에 쉽게 인식할 수 있습니다. 기본 데이터 타입에 대해 논의하기 전에 몇 가지 중요한 점을 언급하겠습니다.

### 숫자 기본 데이터 타입은 부호가 있습니다

Java에서 모든 숫자 기본 데이터 타입은 비트의 시리즈로 표현됩니다. 또한, 이들은 부호가 있습니다. 가장 왼쪽 비트(최상위 비트)는 부호를 나타내며, 1은 음수를 의미하고 0은 양수를 의미합니다.

### 정수 리터럴

리터럴 값은 키보드에서 입력된 값(계산된 값이 아님)을 의미합니다. 정수 리터럴은 다양한 진법 시스템으로 표현될 수 있습니다: 십진수(10진수), 16진수, 8진수, 이진수. 그러나 십진수가 가장 일반적으로 사용되는 표현이라는 것은 놀라운 일이 아닙니다. 정보 제공을 위해, 다음의 모든 선언은 십진수 10을 나타냅니다:

```java
int a = 10; // 십진수, 기본값
int b = 0b1010; // 이진수, 0b 또는 0B로 접두사
int c = 012; // 8진수, 0으로 접두사
int d = 0xa; // 16진수, 0x 또는 0X로 접두사
```

### 부호 비트가 범위에 미치는 영향

부호 비트의 존재는 바이트의 범위가 -27에서 27-1(-128에서 +127 포함)임을 의미합니다. 양수 범위의 -1은 Java에서 0이 양수로 간주되기 때문에 발생합니다. 모든 범위에서 양수 숫자가 하나 줄어드는 것은 아닙니다. 예를 들어, 바이트의 경우 128개의 음수(-1에서 -128)와 128개의 양수(0에서 +127)가 있어 총 256개의 표현(28)이 가능합니다. 이 점을 간단한 예로 강화하면, -1에서 -8까지는 8개의 숫자이고, 0에서 7(포함)까지도 8개의 숫자입니다.

이러한 점을 논의한 후, 다양한 기본 데이터 타입을 살펴보겠습니다.

## Java의 기본 데이터 타입

다음은 이전 표에서 몇 가지 흥미로운 점입니다:

- `byte`, `short`, `char`, `int`, `long`은 정수 값을 가지므로 정수형 타입으로 알려져 있습니다(양수 또는 음수의 정수). 예를 들어, -8, 17, 625는 모두 정수입니다.
  
- `char`는 문자에 사용됩니다. 예를 들어, 'a', 'b', '?', '+'와 같은 문자입니다. 문자 주위에는 단일 인용부호가 있습니다. 코드에서 `char c = 'a';`는 변수 `c`가 문자 'a'를 나타낸다는 의미입니다. 컴퓨터는 모든 문자를 내부적으로 숫자(이진수)로 저장하므로, 문자를 숫자에 매핑하는 인코딩 시스템이 필요합니다. Java는 유니코드 인코딩 표준을 사용하여 모든 문자에 대해 고유한 숫자를 보장합니다. 이 때문에 `char`는 1바이트가 아닌 2바이트를 사용합니다. 사실, 컴퓨터의 관점에서 `char c = 'a';`는 `char c = 97;`과 동일하며, 여기서 97은 유니코드에서 'a'의 십진수 값입니다. 우리는 문자 표현을 선호합니다.

- `short`와 `char`는 모두 2바이트를 필요로 하지만 서로 다른 범위를 가집니다. `short`는 음수를 표현할 수 있지만, `char`는 음수를 표현할 수 없습니다. 반면, `char`는 65,000과 같은 숫자를 저장할 수 있지만, `short`는 이를 저장할 수 없습니다.

- `float`와 `double`은 부동 소수점 숫자에 사용됩니다. 즉, 소수점이 있는 숫자(예: 23.78, -98.453)입니다. 이러한 부동 소수점 숫자는 과학적 표기법을 사용할 수 있습니다. 예를 들어, 130000.0은 `double d1 = 1.3e+5;`로 표현할 수 있으며, 0.13은 `double d2 = 1.3e-1;`로 표현할 수 있습니다.

### 다양한 타입의 표현

이전 내용을 확장하여 정수 리터럴을 다음과 같은 진법 시스템을 사용하여 표현할 수 있습니다:

- **십진수**: 10진수; 숫자 0..9. 기본값입니다.
- **16진수**: 16진수; 숫자 0..9 및 문자 a..f (또는 A..F). 리터럴 앞에 0x 또는 0X를 붙여서 16진수 리터럴임을 나타냅니다.
- **이진수**: 2진수; 숫자 0..1. 리터럴 앞에 0b 또는 0B를 붙여서 이진수 리터럴임을 나타냅니다.

다음은 다양한 진법 시스템을 사용하여 int 변수를 30으로 초기화하는 샘플 코드 조각입니다. 먼저 십진수를 사용하고, 그 다음 16진수, 마지막으로 이진수를 사용합니다:

```java
// 십진수
int dec = 30;
// 16진수 = 16 + 14
int hex = 0x1E;
// 이진수 = 16 + 8 + 4 + 2
int bin = 0b11110;
```

int를 초기화하는 여러 방법이 있지만, 십진수를 사용하는 것이 가장 일반적입니다. 리터럴 숫자(예: 22)는 기본적으로 int로 간주됩니다. 22를 long으로 처리하려면 리터럴에 대문자 또는 소문자 L을 접미사로 붙여야 합니다. 예를 들어:

```java
int x = 10;
long n = 10L;
```

표 2.1에 따르면, int 대신 long을 사용하면 훨씬 더 크고 작은 숫자에 접근할 수 있습니다. long을 나타내기 위해 대문자 L을 사용하는 것이 선호되며, 소문자 l은 숫자 1(하나)과 비슷하기 때문입니다.

부동 소수점 숫자도 유사하게 동작합니다. 십진수는 기본적으로 double입니다. 어떤 십진수를 float로 처리하려면 리터럴에 대문자 또는 소문자 F를 접미사로 붙여야 합니다. 범위가 문제가 되지 않는다면, float를 double 대신 사용하는 이유 중 하나는 메모리 절약입니다( float는 4바이트를 필요로 하고, double은 8바이트를 필요로 합니다). 예를 들어:

```java
double d = 10.34;
float f = 10.34F;
```

char 타입의 변수는 리터럴 주위에 단일 인용부호가 있어야 초기화됩니다. 예를 들어:

```java
char c = 'a';
```

boolean 타입의 변수는 true 또는 false만 저장할 수 있습니다. 이러한 boolean 리터럴은 소문자만 사용되며, Java에서 예약어이므로 대소문자를 구분합니다:

```java
boolean b1 = true;
boolean b2 = false;
```

이로써 Java의 기본 타입 시스템에 대한 섹션을 마칩니다. 여기서는 다양한 타입, 크기/범위, 그리고 몇 가지 코드 조각을 살펴보았습니다. 이제 변수와 기본 타입에 대한 이론을 실제로 적용해 보겠습니다! 그 전에 연습 문제를 도와줄 수 있는 몇 가지 팁을 제공하겠습니다.

### 화면 출력

우리가 알고 있듯이, `System.out.println()`은 괄호 안에 있는 내용을 출력합니다. 연습을 위해 이를 확장해 보겠습니다. 다음은 코드입니다:

```java
String name = "James"; // line 1
int age = 23; // line 2
double salary = 50_000.0; // line 3
String out = "Details: "
    + name
    + ", "
    + age
    + ", "
    + salary; // line 4
System.out.println(out); // line 5
```

- **라인 1**: 문자열 리터럴 "James"를 선언하고 `name` 변수에 초기화합니다. 문자열 리터럴은 이중 인용부호로 묶인 문자(숫자 포함)의 시퀀스입니다. `String` 클래스에 대해서는 12장에서 자세히 다룰 것입니다.
  
- **라인 2와 3**: `int` 타입의 `age`와 `double` 타입의 `salary`를 선언하고 리터럴 값으로 초기화합니다. 라인 3에서 사용된 언더스코어는 큰 숫자를 읽기 쉽게 만듭니다.

- **라인 4**: 출력할 문자열을 구성합니다. 변수의 값과 함께 출력 설명을 추가합니다. Java는 문자열을 왼쪽에서 오른쪽으로 구성합니다. 여기서 `+`는 일반적인 수학적 덧셈이 아닙니다. 문자열 변수나 리터럴이 `+`의 왼쪽 또는 오른쪽에 있을 때, 이 연산은 문자열 추가가 됩니다.

- **라인 5**: `System.out.println(out);`을 통해 최종 문자열을 출력합니다.

Java는 다음과 같은 과정을 수행합니다:

```
"Details: " + "James" => "Details: James"
"Details: James" + ", " => "Details: James, "
"Details: James, " + "23" => "Details: James, 23"
"Details: James, 23" + ", " => "Details: James, 23, "
"Details: James, 23, " + "50000.0" => "Details: James, 23, 50000.0"
```

따라서 "Details: James, 23, 50000.0"이 `out`에 초기화되어, 라인 5를 실행할 때 화면에 표시됩니다.

## 연습 문제

우리의 멋진 공룡 공원에서 모든 것이 잘 진행되고 있습니다. 그러나 몇 가지 관리 작업이 필요합니다:

1. 공원에 있는 공룡을 추적해야 합니다. 주종, 높이, 길이 및 무게를 나타내는 변수를 선언하고 값을 부여한 후 출력하세요.
2. 공룡의 나이, 이름 및 육식인지 여부를 출력하는 프로그램을 작성하세요. 변수에 값을 부여하고 출력하세요.
3. 공원이 바쁠 때가 많습니다. 소방서에서 하루에 허용되는 최대 방문자 수를 도입하라고 조언했습니다. 하루에 허용되는 최대 방문자 수를 나타내는 변수를 선언하고 합리적인 값을 선택하여 출력하세요. "Mesozoic Eden에는 최대 [x]명이 허용됩니다."라는 문장에 출력하세요.
4. 우리 팀은 Mesozoic Eden의 중요한 부분입니다. 직원의 이름과 나이를 나타내는 변수를 선언하고 값을 부여한 후 출력하세요.
5. 공원에 있는 공룡의 수를 알고 싶습니다. 공원에 있는 공룡의 수를 나타내는 변수를 선언하고 값을 부여한 후 출력하세요.
6. 안전이 우리의 최우선입니다. 우리는 안전 등급 척도를 유지하여 기준을 보장합니다. 공원의 안전 등급을 1에서 10까지 나타내는 변수를 선언하고 값을 부여한 후 출력하세요.
7. 이제 공룡 정보를 하나의 문장으로 통합해 보겠습니다. 문자열 연결을 사용하여 공룡의 이름, 나이 및 식단(육식 또는 초식)을 출력하는 프로그램을 작성하세요.
8. 각 공룡 종은 고유한 이름을 가지고 있습니다. 빠른 참조 시스템을 위해 공룡 종의 첫 글자를 나타내는 문자 변수를 선언하고 값을 부여한 후 출력하세요.

### 프로젝트 – 공룡 프로필 생성기

Mesozoic Eden의 일원으로서, 공원에 사는 모든 공룡의 광범위한 데이터베이스를 만드는 임무를 맡았습니다. 지금은 첫 번째 단계인 프로필 생성기를 완성해야 합니다. 이 프로필은 우리의 선사 시대 거주자를 추적하는 데 도움이 될 뿐만 아니라 과학적 연구, 건강 관리, 식단 관리 및 방문자 참여를 위한 필수 데이터를 제공합니다.

이 프로젝트에서는 개별 공룡의 프로필을 모델링하는 프로그램을 개발하는 데 집중할 것입니다. 프로필에는 다음과 같은 특성이 포함되어야 합니다:

- 이름
- 나이
- 종
- 식단 (육식 또는 초식)
- 무게

각 특성은 프로그램 내에서 변수로 저장되어야 합니다. 어떤 공룡을 설명할지 창의력을 발휘해 보세요. 거대한 T-Rex일까요, 아니면 친근한 Stegosaurus일까요? 빠르고 무서운 Velociraptor일 수도 있고, 강력한 Triceratops일 수도 있습니다.

이 변수를 선언하고 값으로 초기화한 후, 프로그램은 공룡의 전체 프로필을 출력해야 합니다. 출력은 "Meet [Name], a [Age]-year-old [Species]. As a [Diet], it has a robust weight of [Weight] kilograms."와 같은 형식이 될 수 있습니다.

### 요약

이번 장에서는 변수가 단순히 이름과 값을 가진 메모리 위치임을 배웠습니다. 변수를 활용하기 위해서는 이를 선언하고 접근하는 방법을 알아야 합니다.

변수를 선언하려면 변수의 이름과 타입을 지정합니다. 예를 들어, `int countOfTitles = 5;`는 `countOfTitles`라는 이름의 int 변수를 5로 선언합니다. 카멜 케이스를 사용하여 이름을 적절히 짓는 것은 코드의 가독성과 유지 관리를 돕는 데 큰 도움이 됩니다. 변수를 접근하려면 변수의 이름을 지정하면 됩니다. 예를 들어, `System.out.println(countOfTitles);`와 같이 사용할 수 있습니다.

Java는 강타입 언어이므로 변수를 선언할 때 변수의 타입을 지정해야 합니다. Java는 사용을 위해 8개의 내장 기본 데이터 타입을 제공합니다. 이들은 소문자로 쉽게 인식할 수 있습니다. 이전 코드에서 `int`는 `countOfTitles` 변수의 기본 데이터 타입입니다. 우리는 기본 타입의 바이트 크기를 살펴보았고, 이는 값의 범위를 결정합니다. 모든 숫자 타입은 부호가 있으며, 가장 왼쪽 비트가 부호를 나타냅니다. `char` 타입은 부호가 없으며, 2바이트 크기로 Java가 전 세계의 모든 언어에서 모든 문자를 지원할 수 있도록 합니다. 코드 조각을 사용하여 다양한 타입의 변수를 사용하는 방법을 보았습니다.

이제 변수를 선언하고 사용하는 방법을 알았으니, 변수를 결합할 수 있는 연산자로 넘어가겠습니다.