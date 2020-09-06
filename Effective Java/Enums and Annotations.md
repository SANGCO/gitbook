

# 아이템 34. int 상수 대신 열거 타입을 사용하라

> In summary, the advantages of enum types over int constants are compelling. Enums are more readable, safer, and more powerful. Many enums require no explicit constructors or members, but others benefit from associating data with each constant and providing methods whose behavior is affected by this data. Fewer enums benefit from associating multiple behaviors with a single method. In this relatively rare case, prefer constant-specific methods to enums that switch on their own values. Consider the strategy enum pattern if some, but not all, enum constants share common behaviors.

> 열거 타입은 확실히 정수 상수보다 뛰어나다. 더 읽기 쉽고 안전하고 강력하다. 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다. 드믈게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다. 이런 열거 타입에서는 switch문 대신 상수별 메서드 구현을 사용하자. 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.



- 열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.
  - `final`로 선언된 변수가 상수이다.
  - 사계절, 태양계의 행성, 카드게임의 카드 종류 등이 좋은 예다.

- 자바에서 열거 타입을 지원하기 전에는 정수 상수를 한 묶음 선언해서 사용하곤 했다.

```java
// 코드 34-1 정수 열거 패턴 - 상당히 취약하다!
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```



- 정수 열거 패턴(int enum pattern) 기법에는 단점이 많다.
  - 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다.
  - 오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자(`==`)로 비교하더라도 컴파일러는 아무런 경고 메시지를 출력하지 않는다.



- 문자열 열거 패턴(string enum pattern)
  - 이 변형은 더 나쁘다.
    - 상수의 의미를 출력할 수 있다는 점은 좋지만, 경험이 부족한 프로그래머가 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 만들기 때문이다.
    - 이렇게 하드코딩한 문자열에 오타가 있어도 컴파일러는 확인할 길이 없으니 자연스럽게 런타임 버그가 생긴다.
    - 문자열 비교에 성능 저하 역시 당연한 결과다.



- 열거 타입(enum type)
  - 자바는 열거 패턴의 단점을 말끔히 씻어주는 동시에 여러 장점을 안겨주는 대안 열거 타입
  - 자바 열거 타입을 뒷받침하는 아이디어는 단순하다.
  - 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다.
  - 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 `final`이다.
    - 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.
    - 열거 타입은 인스턴스 통제된다(9쪽).

```java
// 코드 34-2 가장 단순한 열거 타입
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

- 열거 타입은 컴파일타임 타입 안전성을 제공한다.
  - Apple 열거 타입을 매개변수로 받는 메서드를 선언했다면, 건네받은 참조는 (`null`이 아니라면) Apple의 세 가지 값 중 하나임이 확실하다.
  - 다른 타입의 값을 넘기려 하면 컴파일 오류가 난다.

- 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다.
  - 공개되는 것이 오직 필드의 이름뿐이라, 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다.
  - 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.
- 열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.
  - `Object` 메서드들(3장)을 높은 품질로 구현해놨고, `Comparable`(아이템 14)과 `Serializable`(12장)을 구현했으며, 그 직렬화 형태도 웬만큼 변형을 가해도 문제없이 동작하게끔 구현해놨다.



```java
// 코드 34-3 데이터와 메서드를 갖는 열거 타입
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```

- 가장 단순하게는 그저 상수 모음일 뿐인 열거 타입이지만, (실제로는 클래스이므로) 고차원의 추상 개념 하나를 완벽히 표현해낼 수도 있는 것이다.
  - 태양계의 여덟 행성은 거대한 열거 타입을 설명하기에 좋은 예

- 열거 타입은 근본적으로 불변이라 모든 필드는 `final`이어야 한다(아이템 17).
  - 필드 `public`으로 선언해도 되지만, `private`으로 두고 별도의 `public` 접근자 메서드를 두는 게 낫다(아이템 16).



```java
// 어떤 객체의 지구에서의 무게를 입력받아 여덟 행성에서의 무게를 출력한다.
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      for (Planet p : Planet.values())
         System.out.printf("%s에서의 무게는 %f이다.%n", p, p.surfaceWeight(mass));
   }
}
```

- 열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다.
  - 값들은 선언된 순서로 저장된다.
  - 각 열거 타입 값의 toString 메서드는 상수 이름을 문자열로 반환하므로 `println`과 `printf`로 출력하기에 안성맞춤이다.
- 열거 타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 `private`이나 `package-private` 메서드로 구현한다.
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스(아이템 24)로 만든다.
  - 예를 들어 소수 자릿수의 반올림 모드를 뜻하는 열거 타입인 `java.math.RoundingMode`는 `BigDecimal`이 사용한다.
    - 반올림 모드는 `BigDecimal`과 관련 없는 영역에서도 유용한 개념이라 자바 라이브러리 설계자는 `RoundingMode`를 톱레벨로 올렸다.



- 상수마다 동작이 달라져야 하는 상황도 있다.
  - 사칙연산 계산기의 연산 종류를 열거 타입으로 선언하고, 실제 연산까지 열거 타입 상수가 직접 수행
    - 깨지기 쉬운 코드
      - 새로운 상수를 추가하면 해당 case 문도 추가해야 하는데 깜빡해도 컴파일은 되지만 런타임 오류가 발생 할 수 있다.

```java
// 코드 34-4 값에 따라 분기하는 열거 타입 - 이대로 만족하는가?
public enum Operation {
  	PLUS, MINUS, TIMES, DIVIDE;
  
  	// 상수가 뜻하는 연산을 수행한다.
  	public double apply(double x, double y) {
      	switch(this) {
          	case PLUS:   return x + Y;
          	case MINUS:  return x - Y;
            case TIMES:  return x * Y;
            case DIVIDE:  return x / Y;        
        }
      	throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```



- 열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 클래스 몸체(constant-specific class body), 즉 각 상수에서 자신에 맞게 재정의하는 방법
  - 이를 상수별 메서드 구현(constant-specific method implementation)

```java
// 코드 34-5 상수별 메서드 구현을 활용한 열거 타입
public enum Operation {
    PLUS   {public double apply(double x, double y) { return x + y; }},
    MINUS  {public double apply(double x, double y) { return x - y; }},
    TIMES  {public double apply(double x, double y) { return x * y; }},
    DIVIDE {public double apply(double x, double y) { return x / y; }};

    public abstract double apply(double x, double y);
}
```

- apply 메서드가 상수 선언 바로 옆에 붙어 있으니 새로운 상수를 추가할 때 apply도 재정의해야 한다는 사실을 깜빡하기는 어려울 것이다.
  - 그뿐만 아니라 apply가 추상 메서드이므로 재정의하지 않았다면 컴파일 오류로 알려준다.



```java
// 코드 34-6 상수별 클래스 몸체(class body)와 데이터를 사용한 열거 타입
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y);

    // 코드 34-7 열거 타입용 fromString 메서드 구현하기
    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(toMap(Object::toString, e -> e));

    // 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }

    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values())
            // op를 찍으면 "+", "-" 등 연산 기호가 나온다.
            // 각각의 op.apply()에 x, y 넣으면 해당 연산이 이루어진다. 
            System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

- 상수별 메서드 구현을 상수별 데이터와 결합할 수도 있다.
- 열거 타입의 정적 필드 중 열거 타입의 생성자에서 접근할 수 있는 것은 상수 변수뿐이다(아이템 24).
  - `private final String symbol;`
    - `final` 붙은게 상수
  - 열거 타입 생성자가 실행되는 시점에는 정적 필드들이 아직 초기화되기 전이라, 자기 자신을 추가하지 못하게 하는 제약이 꼭 필요하다.
  - 이 제약의 특수한 예로, 열거 타입 생성자에서 같은 열거 타입의 다른 상수에도 접근할 수 없다.



- 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.

```java
// 코드 34-8 값에 따라 분기하여 코드를 공유하는 열거 타입 - 좋은 방법인가?
public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
  
    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch (this) {
            case SATURDAY: case SUNDAY: // 주말
                overtimePay = basePay / 2;
                break;
            default: // 주중
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 
                  0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
	      }

        return basePay + overtimePay;
    }
}
```

- 상수별 메서드 구현으로 급여를 정확히 계산하는 방법은 두 가지다.
  - 첫째, 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣으면 된다.
  - 둘째, 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 적절히 호출하면 된다.
- 두 방식 모두 코드가 장황해져 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다.



```java
// 코드 34-9 전략 열거 타입 패턴
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

    public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}
```

- 가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 '**전략**'을 선택하도록 하는 것이다.
  - 잔업수당 계산을 `private` 중첩 열거 타입(다음 코드의 PayType)으로 옮기고 PayrollDay 열거 타입의 생성자에서 이 중 적당한 것을 선택
    - PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임



- 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 `switch` 문이 좋은 선택이 될 수 있다.
  - 서드파티에서 가져온 Operation 열거 타입이 있을 때, 각 연산의 반대 연산을 반환하는 메서드

```java
// 코드 34-10 switch 문을 이용해 원래 열거 타입에 없는 기능을 수행한다.
public class Inverse {
    public static Operation inverse(Operation op) {
        switch(op) {
            case PLUS:   return Operation.MINUS;
            case MINUS:  return Operation.PLUS;
            case TIMES:  return Operation.DIVIDE;
            case DIVIDE: return Operation.TIMES;

            default:  throw new AssertionError("Unknown op: " + op);
        }
    }

    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values()) {
            Operation invOp = inverse(op);
            System.out.printf("%f %s %f %s %f = %f%n",
                    x, op, y, invOp, y, invOp.apply(op.apply(x, y), y));
        }
    }
}
```



- 대부분의 경우 열거 타입의 성능은 정수 상수와 별반 다르지 않다. 
  - 열거 타입을 메모리에 올리는 공간과 초기화하는 시간이 들긴 하지만 체감될 정도는 아니다.
- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.
  - 메뉴 아이템, 연산 코드, 명령줄 플래그 등 허용하는 값 모두를 컴파일타임에 이미 알고 있을 때도 쓸 수 있다.
- 열거 타입에 정의된 상수 개수가 영원히 고정불변일 필요는 없다.
  - 열거 타입은 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었다.



## 아이템 35. Ordinal 메서드 대신 인스턴스 필드를 사용하라

- 대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응된다. 
  - 그리고 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 `ordinal`이라는 메서드를 제공한다.



```java
// 코드 35-1 ordinal을 잘못 사용한 예 - 따라 하지 말 것!
public enum Ensemble{
    SOLO, DUET, TRIO, QUARTET, QUINTET
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians(){ return ordinal() + 1; }
}
```

- 동작은 하지만 유지보수하기가 끔찍한 코드다.
  - 상수 선언 순서를 바꾸는 순간 numberOfMusicians가 오동작하며, 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다.

- 해결책은 간단하다. 열거 타입 상수에 연결된 값은 `ordinal` 메서드로 얻지말고, 인스턴스 필드에 저장하자.

```java
// 인스턴스 필드에 정수 데이터를 저장하는 열거 타입
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

- `Enum`의 API 문서를 보면 `ordinal`에 대해 이렇게 쓰여 있다.
  - "대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 `EnumSet`과 `EnumMap` 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다."
    - 따라서 이런 용도가 아니라면 `ordinal` 메서드는 절대 사용하지 말자.



## 아이템 36. 비트 필드 대신 EnumSet을 사용하라

> In summary, **just because an enumerated type will be used in sets, there is no reason to represent it with bit fields.** The EnumSet class combines the con- ciseness and performance of bit fields with all the many advantages of enum types described in Item 34. The one real disadvantage of EnumSet is that it is not, as of Java 9, possible to create an immutable EnumSet, but this will likely be remedied in an upcoming release. In the meantime, you can wrap an EnumSet with Collections.unmodifiableSet, but conciseness and performance will suffer.

> **열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.** EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공하고 아이템 34에서 설명한 열거 타입의 장점까지 선사하기 때문이다. EnumSet의 유일한 단점이라면 (자바 9까지는 아직) 불변 EnumSet을 만들 수 없다는 것이다. 그래도 향후 릴리스에서는 수정되리라 본다. 그때까지는 (명확성과 성능이 조금 희생되지만) Collections.unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다. 



```java
// 코드 36-1 비트 필드 열거 상수 - 구닥다리 기법!
public class Text{
    public static final int STYLE_BOLD          = 1 << 0; //1
    public static final int STYLE_ITALIC        = 1 << 1; //2
    public static final int STYLE_UNDERLINE     = 1 << 2  //4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; //8

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) { ... } 
}
```

- 다음과 같은 식으로 비트별 `OR`를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 **비트 필드**(bit field)라 한다.
  - `text.applyStyles(STYLE_BOLD | STYLE_ITALIC);`
  - 비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.
- 비트 필드는 정수 열거 상수의 단점을 그대로 지니며, 추가로 다음과 같은 문제까지 안고 있다.
  - 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다.
  - 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다.
  - 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입(보통은 `int`나 `long`)을 선택해야 한다.
    - API를 수정하지 않고는 비트 수(32비트 or 64비트)를 더 늘릴 수 없기 때문이다.



```java
// 코드 36-2 EnumSet - 비트 필드를 대체하는 현대적 기법
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    // 사용 예
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

- `java.util` 패키지의 `EnumSet` 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.
  - `Set` 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 `Set` 구현체와도 함께 사용할 수 있다.
- `EnumSet`의 내부는 비트 백터로 구현되었다.
  - 원소가 총 64개 이하라면, 즉 대부분의 경우에 `EnumSet` 전체를 `long` 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.
  - `removeAll`과 `retainAll` 같은 대량 작업은 (비트 필드를 사용할 때 쓰는 것과 같은) 비트를 효울적으로 처리할 수 있는 산술 연산을 써서 구현했다.
  - 그러면서도 비트를 직접 다룰 때 흔히 겪는 오류들에서 해방된다.
    - 난해한 작업을 `EnumSet`이 다 처리해주기 때문이다.
- `EnumSet`은 집합 생성 등 다양한 기능의 정적 팩터리를 제공하는데, 다음 코드에서는 그중 `of` 메서드를 사용했다.
  - `text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));`
  - applyStyles 메서드가 `Enum<Style>`이 아닌 `Set<Style>`을 받은 이유를 생각해보자.
    - 모든 클라이언트가 `EnumSet`을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스로 받는 게 일반적으로 좋은 습관이다(아이템 64).
      - 이렇게 하면 좀 특이한 클라이언트가 다른 Set 구현체를 넘기더라도 처리할 수 있다.



## 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라

> In summary, **it is rarely appropriate to use ordinals to index into arrays: use** **EnumMap** **instead.** If the relationship you are representing is multidimensional, use EnumMap<..., EnumMap<...>>. This is a special case of the general principle that application programmers should rarely, if ever, use Enum.ordinal (Item 35).

> **배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라.** 다차원 관계는 EnumMap<..., EnumMap<...>> 으로 표현하라. "애플리케이션 프로그래머는 Enum.ordinal을 (웬만해서는) 사용하지 말아야 한다(아이템 35)"는 일반 원칙의 특수한 사례다.



```java
// EnumMap을 사용해 열거 타입에 데이터를 연관시키기

// 식물을 아주 단순하게 표현한 클래스
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override public String toString() {
        return name;
    }

    public static void main(String[] args) {
        Plant[] garden = {
            new Plant("바질",    LifeCycle.ANNUAL),
            new Plant("캐러웨이", LifeCycle.BIENNIAL),
            new Plant("딜",      LifeCycle.ANNUAL),
            new Plant("라벤더",   LifeCycle.PERENNIAL),
            new Plant("파슬리",   LifeCycle.BIENNIAL),
            new Plant("로즈마리", LifeCycle.PERENNIAL)
        };

        // 코드 37-1 ordinal()을 배열 인덱스로 사용 - 따라 하지 말 것!
        Set<Plant>[] plantsByLifeCycleArr =
                (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
        for (int i = 0; i < plantsByLifeCycleArr.length; i++)
            plantsByLifeCycleArr[i] = new HashSet<>();
        for (Plant p : garden)
            plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);
        // 결과 출력
        for (int i = 0; i < plantsByLifeCycleArr.length; i++) {
            System.out.printf("%s: %s%n",
                    Plant.LifeCycle.values()[i], plantsByLifeCycleArr[i]);
        }

        // 코드 37-2 EnumMap을 사용해 데이터와 열거 타입을 매핑한다.
        Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
                new EnumMap<>(Plant.LifeCycle.class);
        for (Plant.LifeCycle lc : Plant.LifeCycle.values())
            plantsByLifeCycle.put(lc, new HashSet<>());
        for (Plant p : garden)
            plantsByLifeCycle.get(p.lifeCycle).add(p);
        System.out.println(plantsByLifeCycle);

        // 코드 37-3 스트림을 사용한 코드 1 - EnumMap을 사용하지 않는다!
        System.out.println(Arrays.stream(garden)
                .collect(groupingBy(p -> p.lifeCycle)));

        // 코드 37-4 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑했다.
        System.out.println(Arrays.stream(garden)
                .collect(groupingBy(p -> p.lifeCycle,
                        () -> new EnumMap<>(LifeCycle.class), toSet())));
    }
}
```

- ordinal을 사용한 예는 동작은 하지만 문제가 많다.
  - 특히 가장 심각한 문제는 정확한 정숫값을 사용한다는 것을 여러분이 직접 보증해야 한다는 점이다.
- 훨씬 멋진 해결책이 있다.
  - 코드 37-1에서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 일을 한다.
    - 그러니 Map을 사용할 수도 있을 것이다.
  - 사실 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체가 존재하는데, 바로 EnumMap이 그 주인공이다.
    - 코드 37-2는 코드 37-1을 수정하여 EnumMap을 사용하도록 한 코드다.
  - EnumMap의 성능이 ordinal을 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 때문이다.
    - 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻어낸 것이다.
    - 여기서 EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다(아이템 33).
- 스트림을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작한다.
  - EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.
  - 예컨대 정원에 한해살이와 여러해살이 식물만 살고 두해살이는 없다면, EnumMap 버전에서는 맵을 3개 만들고 스트림 버전에서는 2개만 만든다.



```java
// 코드 37-6 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결했다.
public enum Phase {
    SOLID, LIQUID, GAS;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(
          			// Phase.from을 키로
          			groupingBy(t -> t.from,
                // EnumMap하나 만들고           
                () -> new EnumMap<>(Phase.class),
                // Map에 벨류로 또 맵을 만든다.
                // keyMapper, valueMapper, mergeFunction, mapFactory           
                toMap(t -> t.to, 
                      t -> t, 
                      (x, y) -> y, 
                      () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }

    // 간단한 데모 프로그램 - 깔끔하지 못한 표를 출력한다.
    public static void main(String[] args) {
        for (Phase src : Phase.values()) {
            for (Phase dst : Phase.values()) {
                Transition transition = Transition.from(src, dst);
                if (transition != null)
                    System.out.printf("%s에서 %s로 : %s %n", src, dst, transition);
            }
        }
    }
}
```

```java
public enum Phase {
    // 코드 37-7 EnumMap 버전에 새로운 상태 추가하기
    SOLID, LIQUID, GAS, PLASMA;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
      
      	... // 나머지 코드는 그대로다.
    }
}       
```



## 아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

> In summary, **while you cannot write an extensible enum type, you can emulate it by writing an interface to accompany a basic enum type that implements the interface.** This allows clients to write their own enums (or other types) that implement the interface. Instances of these types can then be used wherever instances of the basic enum type can be used, assuming APIs are written in terms of the interface.

> 열거 타입 자체는 확장할 수 없지만, **인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다.** 이렇게 하면 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입(혹은 다른 타입)을 만들 수 있다. 그리고 API가 (기본 열거 타입을 직접 명시하지 않고) 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다. 



- 열거 타입은 거의 모든 상황에서 이 책 초판에서 소개한 타입 안전 열거 패턴(typesafe enum pattern)보다 우수하다.
  - 단, 예외가 하나 있으니, 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그럴 수 없다는 점이다.
- 실수로 이렇게 설계한 것은 아니다.
  - 사실 대부분 상황에서 열거 타입을 확장하는건 좋지 않은 생각이다.
  - 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 그 반대는 성립하지 않는다면 문제가 있다.
  - 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법도 마땅치 않다.
  - 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 더 복잡해진다.



```java
// 코드 38-1 인터페이스를 이용해 확장 가능 열거 타입을 흉내 냈다.
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

- 다행히 열거 타입으로 이 효과를 내는 멋진 방법이 있다.
  - 기본 아이디어는 열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용하는 것이다.
  - 연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 하면 된다. 
  - 이때 열거 타입이 그 인터페이스의 표준 구현체 역할을 한다.
  - 코드 38-1은 아이템 34읠 Operation 타입을 확장할 수 있게 만든 코드다.

- 열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다.
  - 이렇게 하면 Operation을 구현한 또 다른 열거 타입을 정의해 기본 타입인 BasicOperation을 대체할 수 있다.



```java
// 코드 38-2 확장 가능 열거 타입
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }

//    // 열거 타입의 Class 객체를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예
//    public static void main(String[] args) {
//        double x = Double.parseDouble(args[0]);
//        double y = Double.parseDouble(args[1]);
//        test(ExtendedOperation.class, x, y);
//    }
//    private static <T extends Enum<T> & Operation> void test(
//            Class<T> opEnumType, double x, double y) {
//        for (Operation op : opEnumType.getEnumConstants())
//            System.out.printf("%f %s %f = %f%n",
//                    x, op, y, op.apply(x, y));
//    }

    // 컬렉션 인스턴스를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예
    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        test(Arrays.asList(ExtendedOperation.values()), x, y);
    }
    private static void test(Collection<? extends Operation> opSet,
                             double x, double y) {
        for (Operation op : opSet)
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
}
```

- test() 메서드를 유연하게 변경

  - BasicOperation, ExtendedOperation 등 여러 구현 타입의 연산을 조합해 호출할 수 있게 변경

  - `private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y)`

    - `test(ExtendedOperation.class, x, y);`

  - `Class` 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻이다.

    - 열거 타입이어야 원소를 순회할 수 있고, Operation이어야 원소가 뜻하는 연산을 수행할 수 있기 때문이다.

  - `test(Collection<? extends Operation> opSet, double x, double y)`

    - `test(Arrays.asList(ExtendedOperation.values()), x, y);`
    - `Class` 객체 대신 한정적 와일드카드 타입(아이템 31) 넘기기

    

## 아이템 39. 명명 패턴보다 애너테이션을 사용하라

- 전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다.
- 효과적인 방법이지만 단점도 크다.
  - 오타가 나면 안 된다.
  - 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.
  - 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.



```java
// 코드 39-1 마커(marker) 애너테이션 타입 선언
/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

- 애너테이션이 모든 문제를 해결해주는 멋진 개념으로, JUnit도 버전 4부터 전면 도입하였다.
- 애너테이션 선언에 다는 애너테이션을 메타애너테이션(meta-annotation)이라 한다.
  - `@Retention(RetentionPolicy.RUNTIME)`
    - 이 매타애너테이션은 @Test가 런타임에도 유지되어야 한다는 표시다.
  - `@Target(ElementType.METHOD)`
    - 이 매타애너테이션은 @Test가 반드시 메서드 선언에서만 사용돼야 한다고 알려준다.
      - 따라서 클래스 선언, 필드 선언 등 다른 프로그램 요소에는 달 수 없다.



```java
// 코드 39-2 마커 애너테이션을 사용한 프로그램 예
public class Sample {
    @Test public static void m1() { }        // 성공해야 한다.
    public static void m2() { }
    @Test public static void m3() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m4() { }  // 테스트가 아니다.
    @Test public void m5() { }   // 잘못 사용한 예: 정적 메서드가 아니다.
    public static void m6() { }
    @Test public static void m7() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m8() { }
}

```

```java
// 39-3 마커 애너테이션을 처리하는 프로그램
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
}
```

- @Test 애너테이션이 Sample 클래스의 의미에 직접적인 영향을 주지는 않는다.
  - 그저 이 애너테이션에 관심 있는 프로그램에게 추가 정보를 제공할 뿐이다.
  - 더 넓게 이야기하면, 대상 코드의 의미는 그대로 둔 채 그 애너테이션에 관심 있는 도구에서 틀별한 처리를 할 기회를 준다.
    - 코드 39-3 예제의 RunTests가 바로 그런 도구의 예다.



```java
// 코드 39-4 매개변수 하나를 받는 애너테이션 타입
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

```java
// 코드 39-5 매개변수 하나짜리 애너테이션을 사용한 프로그램
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)
}
```

- 이 애너테이션의 매개변수 타입은 `Class<? extends Throwable>`이다.

- 여기서의 와일드카드 타입은 많은 의미를 담고 있다.
  - "`Throwable`을 확장한 클래스의 `Class` 객체"라는 뜻이며, 따라서 모든 예외(와 오류) 타입을 다 수용한다.
    - 이는 한정적 타입 토큰(아이템 33)의 또 하나의 활용 사례다.
- 코드 39-5를 보면 `class` 리터럴이 애너테이션 매개변수의 값으로 사용됐다.

```java
// 마커 애너테이션과 매개변수 하나짜리 애너태이션을 처리하는 프로그램
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedEx) {
        Throwable exc = wrappedEx.getCause();
        Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
        if (excType.isInstance(exc)) {
            passed++;
        } else {
            System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
        }
    } catch (Exception exc) {
        System.out.println("잘못 사용한 @ExceptionTest: " + m);
    }
}
```





```java
// 코드 39-6 배열 매개변수를 받는 애너테이션 타입
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```

```java
// 배열 매개변수를 받는 애너테이션을 사용하는 프로그램
public class Sample3 {
  
  	...
  
    // 코드 39-7 배열 매개변수를 받는 애너테이션을 사용하는 코드
    @ExceptionTest({ IndexOutOfBoundsException.class,
                     NullPointerException.class })
    public static void doublyBad() {   // 성공해야 한다.
        List<String> list = new ArrayList<>();

        // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
        // NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
}
```

- 배열 매개변수를 받는 애너테이션용 문법은 아주 유연하다.
  - 단일 원소 배열에 최적화했지만, 앞서의 @ExceptionTest들도 모두 수정 없이 수용한다.
  - 원소가 여럿인 배열을 지정할 때는 다음과 같이 원소들을 중괄호로 감싸고 쉼표로 구분해주기만 하면 된다.

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (Throwable wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
        Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
        for (Class<? extends Throwable> excType : excTypes) {
            if (excType.isInstance(exc)) {
                passed++;
                break;
            }
        }
        if (passed == oldPassed)
            System.out.printf("테스트 %s 실패: %s %n", m, exc);
    }
}
```

- 새로운 @ExceptionTest를 지원하도록 테스트 러너를 수정한 모습
  - 꽤 직관적이다.



- 자바 8에서는 여려 개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다.
  - 배열 매개변수를 사용하는 대신 애너테이션에 `@Repeatable` 메타애너테이션을 다는 방식이다.
  - `@Repeatable`을 단 애너테이션은 하나의 프로그램 요소에 여러 번 달 수 있다.

```java
// 코드 39-8 반복 가능한 애너테이션 타입
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

```java
// 반복 가능한 애너테이션의 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

```java
// 반복 가능한 애너테이션을 사용한 프로그램
public class Sample4 {
    
  	...

    // 코드 39-9 반복 가능 애너테이션을 두 번 단 코드
    @ExceptionTest(IndexOutOfBoundsException.class)
    @ExceptionTest(NullPointerException.class)
    public static void doublyBad() {
        List<String> list = new ArrayList<>();

        // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
        // NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
}
```

```java
// 코드 39-10 반복 가능 애너테이션 다루기
if (m.isAnnotationPresent(ExceptionTest.class)
        || m.isAnnotationPresent(ExceptionTestContainer.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (Throwable wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
        ExceptionTest[] excTests =
                m.getAnnotationsByType(ExceptionTest.class);
        for (ExceptionTest excTest : excTests) {
            if (excTest.value().isInstance(exc)) {
                passed++;
                break;
            }
        }
        if (passed == oldPassed)
            System.out.printf("테스트 %s 실패: %s %n", m, exc);
    }
}
```



- 이번 아이템의 테스트 프레임워크는 아주 간단하지만 애너테이션이 명명 패턴보다 낫다는 점은 확실히 보여준다.
  - 테스트는 애너테이션으로 할 수 있는 일 중 극히 일부일 뿐이다.
  - 여러분이 다른 프로그래머가 소스코드에 추가 정보를 제공할 수 있는 도구를 만드는 일을 한다면 적당한 애너테이션 타입도 함께 정의해 제공하자.
  - 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.
- 도구 제작자를 제외하고는, 일반 프로그래머가 애너테이션 타입을 직접 정의할 일은 없다.
  - 하지만 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들을 사용해야 한다(아이템 40, 27).



## 아이템 40. @Override 애너테이션을 일관되게 사용하라

> In summary, the compiler can protect you from a great many errors if you use the Override annotation on every method declaration that you believe to override a supertype declaration, with one exception. In concrete classes, you need not annotate methods that you believe to override abstract method declarations (though it is not harmful to do so).

> 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 여러분이 실수했을 때 컴파일러가 바로 알려줄 것이다. 예외는 한 가지뿐이다. 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우엔 이 애너테이션을 달지 않아도 된다(된다고 해서 해로울 것도 없다).



- @Override는 메서드 선언에만 달 수 있으며, 이 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의했음을 뜻한다.
  - 이 애너테이션을 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방해준다.



```java
// 코드 40-1 영어 알파벳 2개로 구성된 문자열(바이그램)을 표현하는 클래스 - 버그를 찾아보자.
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```

- Set은 중복을 허용하지 않으니 26이 출력될 거 같지만, 실제로는 260이 출력된다.
- Bigram 작성자는 equals 메서드를 재정의하려 한 것으로 보이고(아이템 10) hashCode도 함께 재정의해야 한다는 사실을 잊지 않았다(아이템 11).
  - 그런데 equals를 '재정의(overriding)'한 게 아니라 '다중정의(overloading, 아이템 52)'해버렸다.
    - Object의 equals를 재정의하려면 매개변수 타입을 Object로 해야만 하는데, 그렇게 하지 않은 것이다.



```java
// 버그를 고친 바이그램 클래스
public class Bigram2 {
    private final char first;
    private final char second;

    public Bigram2(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Bigram2))
            return false;
        Bigram2 b = (Bigram2) o;
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram2> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram2(ch, ch));
        System.out.println(s.size());
    }
}
```

- 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너케이션을 달자.
  - 예외는 한 가지뿐이다.
    - 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때는 굳이 @Override를 달지 않아도 된다.
      - 구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아 있다면 컴파일러가 그 사실을 바로 알려주기 때문이다. 



```java
public interface A {

    void test1A();

    default void test2A() {
        System.out.println("test2A()");
    }

}

public class C {

    @Override
    public void test1A() {}

  	// IDE에서 자동으로 붙여준다.
    @Override
    public void test2A() {

    }
}
```

- 디폴트 메서드를 지원하기 시작하면서, 인터페이스 메서드를 구현한 메서드에도 @Override를 다는 습관을 들이면 시그니처가 올바른지 재차 확신 할 수 있다.
- 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 다는 것이 좋다.
  - 상위 클래스가 구체 클래스든 추상 클래스든 마찬가지다.



## 아이템 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

> In summary, marker interfaces and marker annotations both have their uses. If you want to define a type that does not have any new methods associated with it, a marker interface is the way to go. If you want to mark program elements other than classes and interfaces or to fit the marker into a framework that already makes heavy use of annotation types, then a marker annotation is the correct choice. **If you find yourself writing a marker annotation type whose target is ElementType.TYPE, take the time to figure out whether it really should be an annotation type or whether a marker interface would be more appropriate.**

> 마커 인터페이스와 마커 애너테이션은 각자의 쓰임이 있다. 새로 추가하는 메서드 없이 단지 타입 정의가 목적이라면 마커 인터페이스를 선택하자. 클래스나 인터페이스 외의 프로그램 요소에 마킹해야 하거나, 애너테이션을 적극 활용하는 프레임워크의 일부로 그 마커를 편입시키고자 한다면 마커 애너테이션이 올바른 선택이다. **적용 대상이 ElementType.TYPE인 마커 애너테이션을 작성하고 있다면, 잠시 여유를 갖고 정말 애너테이션으로 구현하는 게 옮은지, 혹은 마커 인터페이스가 낫지는 않을지 곰곰이 생각해보자.**



- 아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가점을 표시해주는 인터페이스를 마커 인터페이스(marker interface)라 한다.
- 마커 애너테이션(아이템 39)이 등장하면서 마커 인터페이스는 구식이 되었다는 얘기가 있지만 사실이 아니다.
- 마커 인터페이스는 두 가지 면에서 마커 애너테이션보다 낫다.
  - 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션을 그렇지 않다.
  - 적용 대상을 더 정밀하게 지정할 수 있다.
    - 적용 대상(`@Target`)을 `ElementType.TYPE`으로 선언한 애너테이션은 모든 타입(클래스, 인터페이스, 열거 타입, 애너테이션)에 달 수 있다.
      - 부착 할 수 있는 타입을 더 세밀하게 제한하지는 못한다는 뜻이다.

- 반대로 마커 애너테이션이 마커 인터페이스보다 나은 점으로는 거대한 에너테이션 시스템의 지원을 받는다는 점을 들 수 있다.



- 그렇다면 어떤 때에 마커 애너테이션을, 또 어떤 때에 마커 인터페이스를 써야 하는가?
  - 확실한 것은, 클래스와 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수 등)에 마킹해야 할 때 애너테이션을 쓸 수밖에 없다.
  - 마커를 클래스나 인터페이스에 적용해야 한다면 "이 마킹이 된 객체를 매개변수로 받는 메서드를 작성할 일이 있을까?"라고 자문해보자.
    - 답이 "그렇다"이면 마커 인터페이스를 써야한다.
      - 이렇게 하면 그 마커 인터페이스를 해당 메서드의 매개변수 타입으로 사용하여 컴파일타임에 오류를 잡아낼 수 있다.
    - 이런 메서드를 작성할 일은 절대 없다고 확신한다면 아마도 마커 애너테이션이 나은 선택일 것이다.
  - 추가로, 애너테이션을 활발히 활용하는 프레임워크에서 사용하려는 마커라면 마커 애너테이션을 사용하는 편이 좋을 것이다.



- 이번 아이템은 "타입을 정의할 거라면 인터페이스를 쓰라"고 해석할 수도 있으니, 어떤 의미에서는 "타입을 정의할 게 아니라면 인터페이스를 사용하지 말라"고 한 아이템 22를 뒤집은 것으로 볼 수 있다.