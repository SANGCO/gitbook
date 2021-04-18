

## 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

> In summary, static factory methods and public constructors both have their uses, and it pays to understand their relative merits. Often static factories are preferable, so avoid the reflex to provide public constructors without first considering static factories.

> 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자.



### 정적 팩터리 메서드가 생성자보다 좋은 장점



#### 첫 번째, 이름을 가질 수 있다.

- 생성자를 쓰면 하나의 시그니처에 하나의 생성자만 만들 수 있다.
- 입력 매개변수들의 순서를 다르게해서 이 제한을 피해볼 수도 있지만 이건 실수를 유발할 수 있어서 좋지않은 생각이다.
- 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾸고 각각의 차이를 잘 드러내는 이름을 지어주자.

```java
public BigInteger(int bitLength, int certainty, Random rnd) {
	...
}

// '값이 소수인 BigInteger를 반환한다'는 의미를 더 잘 나타나고 있네.
public static BigInteger probablePrime(int bitLength, Random rnd) {
	...
}
```



#### 두 번째, 호출 될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

- **캐싱**하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
- 플라이웨이트 패턴도 이와 비슷한 기법
- 인스턴스 통제(instance-controlled) 클래스
  - 정적 팩토리 메서드를 사용해서 캐싱한 객체를 반환하게 하면 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있는데 이런 클래스를 인스턴스 통제 클래스라고 한다.
  - 인스턴스를 통제하는 이유는?
    - 싱글턴(singleton; 아이템 3)으로 만들 수 있다.
    - 인스턴스화 불가(noninstantiable; 아이템 4)로 만들 수 있다.
    - 불변 값 클래스(아이템 17)에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.
      - 코드 17-2 생성자 대신 정적 팩터리를 사용한 불변 클래스

```java
public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false); 

public static Boolean valueOf(boolean b) {
   return b ? TRUE : FALSE;
 }
```



#### 세 번째, 반환 타입의 하위 타입 객체를 반환할 수 있는 능력

- 이 능력은 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 '**엄청난 유연성**'을 선물한다.
- API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다.
- 자바 8 이전에는 인터페이스에 정적 메서드를 선언할 수 없어서 이름이 "Type"인 인터페이스를 반환하는 정적 메서드가 필요하면, "Type**s**"라는 (인스턴스화 불가인) 동반 클래스를 만들어 그 안에 정의하는 것이 관례였단다.
  - `java.util.Collection`, `java.util.Collections`
    - Collection 인터페이스에 정적 메서드를 선언할 수 없었어서 나온 동반 클래스(companion class)가 Collections 클래스 이구나.
    - 핵심 인터페이스들에 수정 불가나 동기화 등의 기능을 덧붙인 총 45개의 유틸리티 구현체를 제공 한단다.
      - Collections 까보니 내부에 엄청나게 많은 **private static class**들이 있다.
      - 내부 클래스 구현은 전부 private으로 해서 공개되지 않게 하였고 정적 팩토리 메서드에서는 **인터페이스 타입을 리턴**하고 있다. 
        - API 외견을 훨씬 작게 만듬
        - API를 사용하기 위해 익혀야 하는 개념의 수와 난이도가 낮아졌다.
        - 사용하는 측에서는 인터페이스를 받아서 사용하니 구현 클래스를 몰라도 된다.



#### 네 번째, 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

- 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
- 심지어 다음 릴리스에서는 또 다른 클래스의 객체를 반환해도 된다.
  - 클라이언트는 인터페이스만 보고 가는거니깐 구현체가 바뀌는건 알 수도 없고 알 필요도 없다.

```java
public abstract class EnumSet<E extends Enum<E>> 
  extends AbstractSet<E> implements Cloneable, Serializable {
 
    // 입력 매개변수에 따라 64를 기준으로 RegularEnumSet나 JumboEnumSet을 생성해서 반환한다.
    // 클라이언트가 알고 있는 리턴타입은 EnumSet
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
      Enum<?>[] universe = getUniverse(elementType);
      if (universe == null) {
        throw new ClassCastException(elementType + " not an enum");
      } else {
        return (EnumSet)(universe.length <= 64 ? 
                         new RegularEnumSet(elementType, universe) : 
                         new JumboEnumSet(elementType, universe));
      }
    }

    // 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 수많은 of 정적 팩토리 메서드
    public static <E extends Enum<E>> EnumSet<E> of(E e) {
      EnumSet<E> result = noneOf(e.getDeclaringClass());
      result.add(e);
      return result;
    }

    ...
      
    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4, E e5) { ... }
  
  	public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest) { ... }
  
} 
```



#### 다섯 번째, 정적 패터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

- 이런 유연함은 서비스 제공자 프레임워크(service provider framework)를 만드는 근간이 된다.

  - 서비스 제공자 프레임워크는 클라이언트에게 구현체들을 제공하는 역할을 하면서 클라이언트를 구현체로부터 분리해준다.

  - **서비스 제공자 프레임워크**는 3개의 핵심 컴포넌트로 이뤄진다.

    - 구현체의 동작을 정의하는 **서비스 인터페이스**(service interface)
    - 제공자가 구현체를 등록할 때 사용하는 **제공자 등록 API**(provider registration API)
    - 클라이런트가 서비스의 인터페이스를 얻을 때 사용하는 **서비스 접근 API**(service access API)

  - 3개의 핵심 컴포넌트와 더불어 종종 쓰이는 네 번째 컴포넌트

    - **서비스 제공자 인터페이스**(service provider interface)
      - 이 컴포넌트는 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명해준다.
      - 서비스 제공자 인터페이스가 없으면 서비스 인터페이스의 구현체를 만들 때 리플렉션을 사용해야한다.

  - 브릿지 패턴을 통해 서비스 접근 API는 공급자가 제공하는 것보다 더 풍부한 서비스 인터페이스를 클라이언트에 반환할 수 있다.

  - JDBC (Java **D**ata**b**ase Connectivity)가 대표적인 서비스 제공자 프레임워크

    - `Driver`는 서비스 인터페이스 역할을, `DriverManager.registerDriver`가 제공자 등록 API 역할, `DriverManager.getConnection`이 서비스 접근 API 역할을 수행한다.

      - `Driver` 인터페이스 좀더 살펴보자.

        - `DriverManager` 클래스 안에 `DriverInfo` 클래스 (Wrapper class, 안에 `Driver`  변수를 가진다.)

        - `Driver`는 인터페이스고 구현체는 DB들이 제공해 준다.

        - `Driver` 클래스가 DB와 연동을 해서 `Connection` 클래스(서비스 인터페이스)를 제공해 준다.

        - [Class.forName을 활용해 jdbc driver를 등록하는 과정](https://www.slipp.net/questions/276)

          - Class.forName("package.ClassName")를 실행하는 경우 문자열로 전달되는 클래스가(ex. com.mysql.jdbc.Driver) 존재하는 클래스를 메모리에 로드하는 역할

          - Driver 클래스가 메모리에 로드되면서 static 절이 실행된다.
  
            - 그래서 Driver 클래스만 Class.forName으로 로드 했을 뿐인데 DriverManager는 특정 DB의 드라이버가 있다는 걸 알고 Connection을 맺을 수 있다.
            
            ```java
            public class Driver extends NonRegisteringDriver implements java.sql.Driver {
            		static {
                    try {
                        // static 절에서 Driver를 생성하고 DriverManager에 등록한다. 
                        java.sql.DriverManager.registerDriver(new Driver());
                    } catch (SQLException E) {
                      	throw new RuntimeException("Can't register driver!");
                    }
            		}
              
                 ...
             }
            ```
  
- 자바 1.6 이상부터는 서비스로더 기반으로 JDBC Driver가 자동으로 등록된다.

  - 그래서 Class.forName("com.mysql.jdbc.Driver") 류의 코드를 호출하지 않아도 된다.
    - 내부에 까보니 ServiceLoader 클래스도 사용하고 Class.forName()도 호출하고 있구만.

- ServiceLoader

  - Java6에서 지정된 인터페이스와 일치하는 구현을 검색하고 로드하는 일을 하는 SPI(Service Provider Interface)가 도입됨.

  - SPI는 제 3자가 구현하거나 확장하기 위한 API.

  - JDBC Driver의 경우 SPI에 정의된 인터페이스를 보고 DB 벤더들이 구현해서 제공.

  - ServiceLoader 클래스는 SPI의 핵심 클래스로서 서비스 구현을 로드함.

  - [Java Service Provider Interface](https://www.baeldung.com/java-spi)

```java
public class DriverManager {

  	...
  
    static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }

  	...
  	
    private static void loadInitialDrivers() {
        String drivers;
        try {
            drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
                public String run() {
                    return System.getProperty("jdbc.drivers");
                }
            });
        } catch (Exception ex) {
            drivers = null;
        }
    
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
    
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
 
                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                    // Do nothing
                }
                return null;
            }
        });
    
        println("DriverManager.initialize: jdbc.drivers = " + drivers);
    
        if (drivers == null || drivers.equals("")) {
            return;
        }
        String[] driversList = drivers.split(":");
        println("number of Drivers:" + driversList.length);
        for (String aDriver : driversList) {
            try {
                println("DriverManager.Initialize: loading " + aDriver);
              	// Class.forName() 호출하고 있네.
                Class.forName(aDriver, true, ClassLoader.getSystemClassLoader());
            } catch (Exception ex) {
                println("DriverManager.Initialize: load failed: " + ex);
            }
        }
    }
  
  	...
    
}
```



### 정적 팩터리 메서드 단점



#### 첫 번째, 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.



#### 두 번째, 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

- 생성자처럼 API 설명에 명확히 드러나지 않는다.



### 정적 팩터리 메서드에 흔히 사용하는 명명 방식들


| **명명**                          | 설명                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| **from**                          | 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드 |
| **of**                            | 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드 |
| **valueOf**                       | from과 of의 더 자세한 버전                                   |
| **instance** 혹은 **getInstance** | (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다. |
| **create** 혹은 **newInstance**   | instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다. |
| **getType**                       | getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다. |
| **newType**                       | newInstance와 같으나, **생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.** `BufferedReader br = Files.newBufferedReader(path);` |
| **type**                          | getType과 newType의 간결한 버전                              |



## 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

> In summary, the Builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters, especially if many of the parameters are optional or of identical type. Client code is much easier to read and write with builders than with telescoping constructors, and builders are much safer than JavaBeans.

> 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 더 낫다. 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.



### 점층적 생성자 패턴

- 점층적 생성자 패턴도 쓸 수 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.
- 클라이언트가 실수로 매개변수의 순서를 바꿔 건네줘도 컴파일러는 알아채지 못하고, 결국 런타임에 엉뚱한 동작을 하게 된다(아이템 51)

```java
// 코드 2-1 점층적 생성자 패턴 - 확장하기 어렵다!
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)  필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
    
}
```



### 자바빈즈 패턴

- 자바빈즈 패턴에서는 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기  전까지는 **일관성(consistency)이 무너진 상태**에 놓이게 된다.
- 일관성이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변(아이템 17)으로 만들 수 없으며 스레드 안전성을 얻으려면 프로그래머가 추가 작업을 해줘야한다.
  - **불변(immutable 혹은 immutability)**은 어떠한 변경도 허용하지 않는다는 뜻
    - 대표적으로 `String` 객체는 한번 만들어지면 절대 값을 바꿀 수 없는 불변 객체다.
  - **불변식(invariant)**은 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건을 말한다.
    - 기간을 표현하는 `Period` 클래스에서 start 필드의 값은 반드시 end 필드의 값보다 앞서야 하므로, 두 값이 역전되면 역시 불변식이 깨진 것이다(아이템 50 참조). 
  - 가변 객체에도 불변식은 존재할 수 있으며, 넓게 보면 불변은 불변식의 극단적인 예라 할 수 있다.

```java
// 코드 2-2 자바빈즈 패턴 - 일관성이 깨지고, 불변으로 만들 수 없다.
public class NutritionFacts {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private int servingSize  = -1; // 필수; 기본값 없음
    private int servings     = -1; // 필수; 기본값 없음
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```



### 빌더 패턴

- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.
  - 추상 클래스는 추상 빌더를, 구체 클래스(concrete class)는 구체 빌더를 갖게 한다.
- Pizza.Builder 클래스는 재귀적 타입 한정(아이템 30)을 이용하는 제네릭 타입이다.
  - 여기에 추상 메서드인 self를 더해 하위 클래스에서는 형변환하지 않고도 메서드 연쇄를 지원할 수 있다.
    - self 타입이 없는 자바를 위한 이 우회 방법을 **시뮬레이트한 셀프 타입(simulated self-type) 관용구**라 한다.
    - 추상 클래스에 addTopping 메서드를 보면 self()라는 추상 메서드를 리턴하고 있다.
- 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능을 **공변 반환 타이핑(covariant return typing)**이라 한다.
  - 이 기능을 이용하면 클라이언트가 형변환에 신경 쓰지 않고도 빌더를 사용할 수 있다.

```java
// 코드 2-4 계층적으로 설계된 클래스와 잘 어울리는 빌더 패턴

// 참고: 여기서 사용한 '시뮬레이트한 셀프 타입(simulated self-type)' 관용구는
// 빌더뿐 아니라 임의의 유동적인 계층구조를 허용한다.

public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        
      	public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

      	// 외부에서 호출할 수 있게 접근 제한자가 public이 되야겠지.
        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        // self() 호출은 여기서 구현체는 하위 클래스에, 그래서 접근 제한자가 protected
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }
}
```

```java
// 코드 2-5 뉴욕 피자 - 계층적 빌더를 활용한 하위 클래스
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    @Override public String toString() {
        return toppings + "로 토핑한 뉴욕 피자";
    }
}
```

```java
// 코드 2-6 칼초네 피자 - 계층적 빌더를 활용한 하위 클래스
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() { return this; }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }

    @Override public String toString() {
        return String.format("%s로 토핑한 칼초네 피자 (소스는 %s에)",
                toppings, sauceInside ? "안" : "바깥");
    }
}
```

```java
// 계층적 빌더 사용
public class PizzaTest {
    public static void main(String[] args) {
        // 정적 중첩 클래스 객체 생성 방법 (.Inner()가 생성자임)
      	// Outer.Inner inner = new Outer.Inner();
      	// addTopping()은 self() 반환
        // sauceInside()는 this 반환
       	// 하위 클래스에서 메서드 체이닝 가능 
        NyPizza pizza = new NyPizza.Builder(SMALL)
                .addTopping(SAUSAGE).addTopping(ONION).build();
        Calzone calzone = new Calzone.Builder()
                .addTopping(HAM).sauceInside().build();
        
        System.out.println(pizza);
        System.out.println(calzone);
    }
}
```



- 빌더 패턴은 상당히 유연하다.
  - 빌더 하나로 여러 객체를 순회하면서 만들 수 있다.
  - 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.
  - 객체마다 부여되는 일련번호와 같은 특정 필드는 빌더가 알아서 채우도록 할 수도 있다.

- 빌더 패턴을 사용할 때 체크
  - 빌더 생성 비용이 크지 않지만 성능에 민감한 상황에서는 문제가 될 수 있다.
  - 매개변수가 4개 이상은 되어야 빌터 패턴이 값어치를 한다.
    - API는 시간이 지날수록 매개변수가 많아지는 경향이 있음을 명심하자.
    - 생성자나 정적 팩터리 방식으로 시작했다가 나중에 매개변수가 많아지면 빌더 패턴으로 전환할 수도 있지만, 이전에 만들어둔 생성자와 정적 팩터리가 아주 도드라져 보일 것이다.
      - 애초에 빌더로 시작하는 편이 나을 때가 많단다.



### 네스티드 클래스

- `new NyPizza.Builder(SMALL)` 생소한 모양이네.
- 네스티드 클래스 (멤버 클래스)
  - Static 네스티드 클래스
    - `new Outer.Nested();`
    - 이름에 static이 붙었다고 생성되어 있다고 생각하면 안된다.
      - Outer 클래스의 static 변수를 공유 할 수 있다.
    - Outer 클래스를 생성안하고 Static 네스티드 클래스만 생성 할 수 있다.
  - Non-static 네스티드 클래스 (이너 클래스)
    - `new Outer.new Nested();`
    - new가 두개네
      - Outer 클래스를 생성안하고 네스티드 클래스만 생성 할 수 없다.
  - Non-static 네스티드 클래스는 **숨은 외부 참조**가 있으니 멤버 클래스는 되도록 static으로 만들어라.(아이템 24)
    - `Outer.this.method()`
      - 클래스명.this로 네스티드 클래스에서 Outer 클래스 객체에 접근 할 수 있다.
  - 멤버 클래스는 클래스의 정의를 감추어야 할 때 유용하게 사용이 된다.



## 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

- 싱글턴(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
- 싱글턴의 정형적인 예로는 함수(아이템 24)와 같은 무상태(stateless) 객체나 설계상 유일해야 하는 시스템 컴포넌트를 들 수 있다.
  - 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.



### public static final 필드 방식의 싱글턴

- `private` 생성자는 `public static final` 필드인 Elvis 타입 INSTANCE 필드를 초기화 할 때 딱 한 번만 호출된다.
  - 예외는 단 한가지, 권한이 있는 클라이언트는 리플렉션 API(아이템 65)인 `AccessibleObject.setAccessible`을 사용해 private 생성자를 호출할 수 있다.
- 장점
  - 해당 클래스가 싱클턴임이 API에 명백히 들어난다.
  - 간결하다.

```java
// 코드 3-1 public static final 필드 방식의 싱글턴
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```



### 정적 팩터리 방식의 싱글턴

- 장점
  - API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점
  - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 점(아이템 30)
  - 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다는 점
    - `Elvis::getInstancce`를 `Supplier<Elvis>`로 사용할 수도 있다.

```java
// 코드 3-2 정적 팩터리 방식의 싱글턴
public class Elvis {
  	// 접근 제한자가 private임. 
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();
    }
}
```



### 직렬화

- 앞의 둘 중 하나의 방식으로 만든 클래스를 직렬화하려면(12장 참조) 단순히 `Serializable`을 구현한다고 선언하는 것만으로는 부족하다.
- 모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve 메서드를 제공해야 한다(아이템89). 



### 열거 타입 방식의 싱글턴 - 바람직한 방법

- `public` 필드 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화할 수 있고, 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.
  - 대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.
  - 단, 만들려는 싱글턴이 `Enum` 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다. 

```java
// 열거 타입 방식의 싱글턴 - 바람직한 방법
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("기다려 자기야, 지금 나갈께!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```



## 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

- 이따금 단순히 정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있을 것이다.
  - `java.lang.Math`와 `java.util.Arrays`처럼 기본 타입 값이나 배열 관련 메서드들을 모아놓을 수 있다.
  - `java.util.Collections`처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드(혹은 팩터리)를 모아놓을 수도 있다(자바 8부터는 이런 메서드를 인터페이스에 넣을 수 있다).
  - `final` 클래스와 관련한 메서들을 모아놓을 때도 사용한다.
    - `final` 클래스를 상속해서 하위 클래스에 메서드를 넣는건 불가능하기 때문이다.
- 정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한게 아니다. 하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다.
  - 실제로 공개된 API들에서도 이처럼 의도치 않게 인스턴스화할 수 있게 된 클래스가 종종 목격되곤 한다.
- 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.
  - 하위 클래스를 만들어 인스턴스화하면 그만이다.
  - 오히려 추상 클래스를 보고 사용자가 상속해서 쓰라는 뜻으로 오해할 수 있으니 더 큰 문제다.
- private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.
  - 생성자가 존재하는데 호출할 수 없으니 그다지 직관적이지 않다. 앞에 적절한 주석을 달아두자.
  - 이 방식은 상속을 불가능하게 하는 효과도 있다. 

```java
// 코드 4-1 인스턴스를 만들 수 없는 유틸리티 클래스
public class UtilityClass {
    // 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용).
    private UtilityClass() {
        throw new AssertionError();
    }

    ...
      
}
```



## 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

> In summary, do not use a singleton or static utility class to implement a class that depends on one or more underlying resources whose behavior affects that of the class, and do not have the class create these resources directly. Instead, pass the resources, or factories to create them, into the constructor (or static factory or builder). This practice, known as dependency injection, will greatly enhance the flexibility, reusability, and testability of a class.

> 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안 된다. 대신에 필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에(혹은 정적 팩터리나 빌더에) 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다.



- 사용하는 자원에 따라 동작이 달라지는 클래스
  - 맞춤법 검사기는 사전에 의존
    - 사전이 달라지면 동작이 달라지겠지?
  - 잘못 사용한 예
    - 정적 유틸리티
      - 생성자 `private`으로 만들고 `isValid()`, `suggestions()`  메서드는 `static`으로
    - 싱글턴

- 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식
  - 이 패턴의 쓸만한 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있다.
  - 팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다.
    - 팩터리 메서드 패턴을 구현
    - 자바 8에서 소개한 `Supplier<T>` 인터페이스가 팩터리를 표현한 완벽한 예다.
      - `Supplier<T>`를 입력으로 받은 메서드는 일반적으로 한정적 와일드 카드 타입(bounded wildcard type, 아이템 31)을 사용해 팩터리의 타입 매개변수를 제한해야 한다.
      - `Mosaic create(Supplier<? extends Tile> tileFactory) {...}`



## 아이템 6. 불필요한 객체 생성을 피하라

- 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다. 
  - 재사용은 빠르고 세련되다. 
  - 특히 불변 객체(아이템 17)는 언제든 재사용할 수 있다.

```java
String s = new String("bikini"); // 따라 하지 말 것!
// 이 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다.
String s = "bikini";
```



- 생성자 대신 정적 팩터리 매서드(아이템 1)를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다. 
  - 예컨대 `Boolean(String)` 생성자 대신 `Boolean.valueOf(String)` 팩터리 메서드를 사용하는 것이 좋다.

```java
public static Boolean valueOf(String s) {
  return parseBoolean(s) ? TRUE : FALSE;
}

public static boolean parseBoolean(String s) {
  return "true".equalsIgnoreCase(s);
}
```



- 불변 객체만이 아니라 가변 객체라 해도 사용 중에 변경되지 않을 것임을 안다면 재사용할 수 있다.

- 생성 비용이 아주 비싼 객체도 더러 있다. 이런 '비싼 객체'가 반복해서 필요하다면 **캐싱하여 재사용**하길 권한다.
  - `static final`를 써서 클래스 초기화 과정에서 초기화 되도록 만들었군.

```java
// 값비싼 객체를 재사용해 성능을 개선한다.
public class RomanNumerals {
    // 코드 6-1 성능을 훨씬 더 끌어올릴 수 있다!
    static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }

    // 코드 6-2 값비싼 객체를 재사용해 성능을 개선한다.
    // 클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱해뒀다가 재사용
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```



- `Map` 인터페이스의 `keySet` 메서드를 호출할 때마다 새로운 `Set` 인스턴스가 만들어지리라고 순진하게 생각할 수도 있지만, 사실은 매번 같은 `Set` 인스턴스를 반환할지도 모른다.

```java
Map<String, String> map = new HashMap<>();
map.put("1", "111");
map.put("2", "222");
map.put("3", "333");

// strings1, 2, 3 다 메모리 주소가 같은 동일한 인스턴스임.
Set<String> strings1 = map.keySet();
Set<String> strings2 = map.keySet();
Set<String> strings3 = map.keySet();

// add는 안된다.
// 여기서 반환 되는 Set은 HashMap 안에 이너클래스 KeySet이다.
// strings1.add("4");
strings1.remove("1");
```

```java
public class HashMap<K,V> extends AbstractMap<K,V> 
  												implements Map<K,V>, Cloneable, Serializable {
  	
  	...
  
  	public Set<K> keySet() {
        Set<K> ks = keySet;
      	// keySet이 null일 때만 생성해서 반환 
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }

  	// 반환하는 Set은 HashMap 안에 이너 클래스 KeySet이다.
    // size()를 달라고 하면 HashMap에 있는 size를 그냥 리턴하네.
    // add() 이런건 없다.
    final class KeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<K> iterator()     { return new KeyIterator(); }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        
      	...
    }
  
    ...
      
}  
```



- 불필요한 객체를 만들어내는 또 다른 예로 오토박싱(auto boxing)을 들 수 있다.
  - 오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다.
  - **박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.**

```java
// 코드 6-3 끔찍이 느리다! 객체가 만들어지는 위치를 찾았는가?
public class Sum {
    private static long sum() {
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
    	  // long 타입인 i가 Long 타입인 sum에 더해질 때마다 불필요한 Long 인스턴스가 만들어 지고 있다.
        return sum;
    }
}
```



- 이번 아이템을 "객체 생성은 비싸니 피해야 한다"로 오해하면 안 된다. 
  - 특히나 요즘의 JVM에서는 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일이 크게 부담되지 않는다. 
  - 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이다.
- 이번 아이템이 **"기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라"**라면, 아이템 50은 **"새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라"**다.
  - **방어적 복사**가 필요한 상황에서 객체를 재사용했을 때의 피해가, 필요 없는 객체를 반복 생성했을 때의 피해보다 훨씬 크다는 사실을 기억하자.



## 아이템 7. 다 쓴 객체 참조를 해제하라

> Because memory leaks typically do not manifest themselves as obvious failures, they may remain present in a system for years. They are typically discovered only as a result of careful code inspection or with the aid of a debugging tool known as a *heap profiler*. Therefore, it is very desirable to learn to anticipate problems like this before they occur and prevent them from happening.


> 메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.



### 메모리 누수

- 코드 7-1에서 메모리 누수는 어디서 일어날까? 이 코드에서는 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 프로그램에서 그 객체들을 더 이상 사용하지 않더라도 말이다.
  - 이 스택이 그 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다.
  - **객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체(그리고 또 그 객체들이 참조하는 모든 객체...)를 회수해가지 못한다.** 그래서 단 몇 개의 객체가 매우 많은 객체를 회수되지 못하게 할 수 있고 잠재적으로 성능에 악영향을 줄 수 있다.

```java
// 코드 7-1 메모리 누수가 일어나는 위치는 어디인가?
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

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        // 메모리 누수가 일어나는 부분!
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

```java
// 코드 7-2 제대로 구현한 pop 메서드 (37쪽)
public Object pop() {
    if (size == 0)
    	  throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```



- 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.
  - 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 **유효 범위(scope) 밖으로** 밀어내는 것이다. 
  - 변수의 범위를 최소가 되게 정의했다면(아이템 57) 이 일은 자연스럽게 이뤄진다.
- 일반적으로 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.
- **캐시** 역시 메모리 누수를 일으키는 주범이다.
- 메모리 누수의 세 번째 주범은 바로 **리스너(listener) 혹은 콜백(callback)**이라 부르는 것이다.



## 아이템 8. finalizer와 cleaner 사용을 피하라

> In summary, don’t use cleaners, or in releases prior to Java 9, finalizers, except as a safety net or to terminate noncritical native resources. Even then, beware the indeterminacy and performance consequences.


> cleaner(자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.



### 객체 소멸자에 대해서

- 자바가 제공하는 객체 소멸자는 2개 `finalizer`, `cleaner`
- finalize() 메서드를 오버라이드하면 가비지 컬렉션의 대상이 된다.
  - `Object` 클래스에 finalize() 메서드가 있다.
- `Cleaner` 클래스로 객체 청소하기
  - `Cleaner cleaner = Cleaner.create();`
  - `cleaner.register(object, runnableInstance);`
    - object는 가비지 컬렉션을 위해 모니터일 되는 대상이고 runnableInstance는 수행 되어야 할 액션
    - [Java 9 garbage collection cleaner class with example](https://codippa.com/java-9-cleaner-example-for-garbage-collection/)
  - 스레드는 등록 된 모든 청소 조치가 완료되고 가비지 콜렉터가 클리너 자체를 회수 할 때까지 실행된다.
  - 가비지 컬렉션을 호출하니깐 `cleaner`가 실행될 때도 있고 아닐 때도 있네. (밑에 예제 참고)
- `finalizer`, `cleaner`로 객체를 소멸 시킬 수 있으니 사용하자가 이 장에 핵심이 아니다. 
- 명시적으로 clean 수행
  -  **`AutoCloseable`을 구현해서 `try-with-resource`를 사용해서 `try-catch` 문이 끝나면 명시적으로 종료**
     -  `AutoCloseable` close() 메소드 재정의



### finalizer와 cleaner 문제점들

- `finalizer`는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
  - 자바 9에서는 `finalizer`를 사용 자제(deprecated) API로 지정하고 `cleaner`를 그 대안으로 소개했다.
- `cleaner`는 `finalizer`보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.
- 프로그램 생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 `finalizer`나 `cleaner`에 의존해서는 안 된다.
- `finalizer`와 `cleaner`는 심각한 성능 문제도 동반한다.
- `finalizer`를 사용한 클래스는 `finalizer` 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
- 파일이나 스레드 등 종료해야 할 자원을 담고 있는 객체의 클래스에서 `finalizer`와 `cleaner`를 대신해줄 묘안은 무엇일까?
  - 그저 `AutoCloseable`을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 `close` 메서드를 호출하면 된다.
    - 일반적으로 예외가 발생해도 제대로 종료되도록 `try-with-resources`를 사용해야 한다.



### finalizer와 cleaner는 대체 어디에 쓰는 건가?

- 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 **안전망 역할**이다.
  - `finalizer`나 `cleaner`가 즉시 (혹은 끝까지) 호출되리라는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 아예 안 하는 것보다는 나으니 말이다.
    - 안전망 역할로 사용할지 말지는 그럴만한 값어치가 있는지 심사숙고해서 결정하자.
  - 자바 라이브러리의 일부 클래스는 안전망 역할의 `finalizer`를 제공한다.
    - `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`가 대표적이다.
- 두 번째 예는 네이티브 피어(native peer)와 연결된 객체에서다.
  - 네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다.
  - 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지 못한다.
    - cleaner나 finalizer가 나서서 처리하기에 적당한 작업
  - 성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 앞서 설명한 close 메서드를 사용해야 한다.
- `cleaner`는 사용하기에 조금 까다롭다.
  - `AutoCloseable`, close()
    - `try-with-resources`에서 try() 괄호 안에는 `AutoCloseable` 구현체만 들어갈 수 있다.
    - `try-catch` 절이 끝나면 close() 메서드를 호출해 준다. 
  - `Runnable`, run()
    - 자바에서는 스레드를 만들때 Thread 클래스와 `Runnable` 인터페이스를 사용할 수 있다.
  - `Cleaner`, `Cleanable`, clean()
    - `Cleaner`를 사용하면 `Runnable`의 run() 메서드에서 자원해제 작업을 해준다.
      - 적당한 예제를 찾기 힘드네.

```java
// 코드 8-1 cleaner를 안전망으로 활용하는 AutoCloseable 클래스
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    // 그래서 정적 중첩 클래스를 썼다. 
    // 정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖게된다.(아이템 24)
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출한다.
        // close 메서드가 내부에서 clean()을 호출 or cleaner가 clean()을 호출
        // clean() 하면 run()이 실행되는 부분은 밑에 코드 참조
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```

```java
// cleaner 안전망을 갖춘 자원을 제대로 활용하는 클라이언트
public class Adult {
    public static void main(String[] args) {
        // try-with-resources를 써서 AutoCloseable에 close()가 항상 호출된다.
        // 이렇게 잘쓰면 안전망으로 만든 Room의 cleaner를 쓸일이 없다.
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```

```java
// cleaner 안전망을 갖춘 자원을 제대로 활용하지 못하는 클라이언트
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("Peace out");

        // 다음 줄의 주석을 해제한 후 동작을 다시 확인해보자.
      	// System.gc()를 하면 Cleaner가 동작할 때도 있고 안할 때도 있다. 예측할 수 없다!!!
        // 단, 가비지 컬렉러를 강제로 호출하는 이런 방식에 의존해서는 절대 안 된다!
//      System.gc();
    }
}
```

```java
// Cleaner에 register에 Runnable 구현체를 던지면 Runnable에 run()을 호출 해주는 부분.

public final class Cleaner {
  	
		public Cleaner.Cleanable register(Object obj, Runnable action) {
        Objects.requireNonNull(obj, "obj");
        Objects.requireNonNull(action, "action");
        return new PhantomCleanableRef(obj, this, action);
    }
  	
}

public final class CleanerImpl implements Runnable {

  	public static final class PhantomCleanableRef extends PhantomCleanable<Object> {
        private final Runnable action;

        public PhantomCleanableRef(Object obj, Cleaner cleaner, Runnable action) {
            super(obj, cleaner);
            this.action = action;
        }

        PhantomCleanableRef() {
            this.action = null;
        }

        protected void performCleanup() {
            this.action.run();
        }

        public Object get() {
            throw new UnsupportedOperationException("get");
        }

        public void clear() {
            throw new UnsupportedOperationException("clear");
        }
    }
  
}
```



## 아이템 9. try-finally보다는 try-with-resources를 사용하라

> The lesson is clear: Always use try-with-resources in preference to try- finally when working with resources that must be closed. The resulting code is shorter and clearer, and the exceptions that it generates are more useful. The try- with-resources statement makes it easy to write correct code using resources that must be closed, which was practically impossible using try-finally.


> 꼭 회수해야 하는 자원을 다룰 때는 try-finall 말고, try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는  정확하고 쉽게 자원을 회수할 수 있다.



- 자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.
  - `InputStream`, `OutputStream`, `java.sql.Connection`
- `try-with-resources`를 사용하려면 해당 자원이 `AutoCloseable` 인터페이스를 구현해야 한다.
  - 자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스가 이미 `AutoCloseable`을 구현하거나 확장해뒀다.
  - 클래스를 만들 때 닫아야하는 자원이라면 `AutoCloseable`을 반드시 구현해야 한다.
- `try-with-resources` 구문을 사용하게 되면 입출력 처리시 예외가 발생하는 경우 JVM이 자동으로 close()를 호출하여 자원을 반납시켜줍니다.

