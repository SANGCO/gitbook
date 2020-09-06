

## 아이템 42. 익명 클래스보다는 람다를 사용하라

> In summary, as of Java 8, lambdas are by far the best way to represent small function objects. **Don’t use anonymous classes for function objects unless you have to create instances of types that aren’t functional interfaces.** Also, remember that lambdas make it so easy to represent small function objects that it opens the door to functional programming techniques that were not previously practical in Java.

> 자바가 8로 판올림되면서 작은 함수 객체를 구현하는 데 적합한 람다가 도입되었다. **익명 클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들 때만 사용하라.** 람다는 작은 함수 객체를 아주 쉽게 표현할 수 있어 (이전 자바에서는 실용적이지 않던) 함수형 프로그래밍의 지평을 열였다.



- 예전에는 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용했다.
  - 이런 인터페이스의 인스턴스를 함수객체(function object)라고 하여, 특정 함수나 동작을 나타내는 데 썼다.



```java
// 코드 42-1 익명 클래스의 인스턴스를 함수 객체로 사용 - 낡은 기법이다!
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

- 문자열을 길이순으로 정렬하는데, 정렬을 위한 비교 함수로 익명 클래스를 사용한다.
- 전략 패턴처럼, 함수 객체를 사용하는 과거 객체 지향 디자인 패턴에는 익명 클래스면 충분했다.
  - 하지만 익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았다.
- 자바 8에 와서 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 특별한 대우를 받게 되었다.
  - 지금은 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 람다식(lambda expression, 혹은 짧게 람다)을 사용해 만들 수 있게 된 것이다.
    - 람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다.

```java
// 코드 42-2 람다식을 함수 객체로 사용 - 익명 클래스 대체
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

- 여기서 람다, 매개변수(s1, s2), 반환값의 타입은 각각 (Comparator<String>), String, int지만 코드에서는 언급이 없다.
  - 우리 대신 컴파일러가 문맥을 살펴 타입을 추론해준 것이다.
- 타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.
  - 그런 다음 컴파일러가 "타입을 알 수 없다"는 오류를 낼 때만 해당 타입을 명시하면 된다.
  - 반환값이나 람다식 전체를 형변환해야 할 때도 있겠지만, 아주 드물 것이다.

```java
// 람다 자리에 비교자 생성 메서드를 사용
Collections.sort(words, comparingInt(String::length));
```

```java
// 자바 8 때 List 인터페이스에 추가된 sort 메서드를 이용하면 더욱 짧아진다.
words.sort(comparingInt(String::length));
```

- 람다를 언어 차원에서 지원하면서 기존에는 적합하지 않았던 곳에서도 함수 객체를 실용적으로 사용할 수 있게 되었다.



- 아이템 34에서는 상수별 클래스 몸체를 구현하는 방식보다는 열거 타입에 인스턴스 필드를 두는 편이 낫다고 했다.
  - 람다를 이용하면 후자의 방식, 즉 열거 타입의 인스턴스 필드를 이용하는 방식으로 상수별로 다르게 동작하는 코드를 쉽게 구현할 수 있다.

```java
// 코드 42-3 상수별 클래스 몸체와 데이터를 사용한 열거 타입
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
}
```

```java
// 코드 42-4 함수 객체(람다)를 인스턴스 필드에 저장해 상수별 동작을 구현한 열거 타입
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

- 람다 기반 Operation 열거 타입을 보면 상수별 클래스 몸체는 더 이상 사용할 이유가 없다고 느낄지 모르지만, 꼭 그렇지는 않다.
  - 메서드나 클래스와 달리, 람다는 이름이 없고 문서화도 못 한다.
  - 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
  - 람다는 한 줄일 때 가장 좋고 길어야 세 줄 안에 끝내는 게 좋다.

- 열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일타임에 추론된다.
  - 따라서 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다(인스턴스는 런타임에 만들어지기 때문이다).
  - 따라서 상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 몸체를 사용해야 한다.
- 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 써야 한다.
  - 비슷하게 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 쓸 수 있다.
- 람다는 자신을 참조할 수 없다.
  - 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다.
    - 반면 익명 클래스에서의 this는 익명 클래스의 인스턴스 자신을 가리킨다.
    - 그래서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.
- 람다도 익명 클래스처럼 직렬화 형태가 구현별로(가령 가상머신별로) 다를 수 있다. 
  - 따라서 람다를 질렬화하는 일은 극히 삼가야 한다(익명 클래스의 인스턴스도 마찬가지다).
  - 직렬화해야만 하는 함수 객체가 있다면(가령 Comparator처럼) private 정적 중첩 클래스(아이템 24)의 인스턴스를 사용하자.



## 아이템 43. 람다보다는 메서드 참조를 사용하라

> In summary, method references often provide a more succinct alternative to lambdas. **Where method references are shorter and clearer, use them; where they aren’t, stick with lambdas.**

> 메서드 참조는 람다의 간단명료한 대안이 될 수 있다. **메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하라.**



- 람다가 익명 클래스보다 나은 점 중에서 가장 큰 특징은 간결함이다.
  - 그런데 자바에는 함수 객체를 심지어 람다보다도 더 간결하게 만드는 방법이 메서드 참조(method reference)다.
- `map.merge(key, 1, (count, incr) -> count + incr);`
  - merge 메서드는 키, 값, 함수를 인수로 받으며, 주어진 키가 맵 안에 아직 없다면 주어진 {키, 값} 쌍을 그대로 저장한다.
  - 반대로, 키가 이미 있다면 (세 번째 인수로 받은) 함수를 현재 값과 주어진 값에 적용한 다음, 그 결과로 현재 값을 덮어쓴다.
    - 즉, 맵에 {키, 함수의 결과} 쌍을 정장한다.
  - 람다 대신 이 메서드의 참조를 전달하면 똑같은 결과를 더 보기 좋게 얻을 수 있다.
    - `map.merge(key, 1, Integer::sum);`



- 람다로 구현했을 때 너무 길거나 복잡하다면 메서드 참조가 좋은 대안이 되어준다.
  - 람다로 작성할 코드를 새로운 메서드에 담은 다음, 람다 대신 그 메서드 참조를 사용하는 식이다.
    - 메서드 참조에는 기능을 잘 드러내는 이름을 지어줄 수 있고 친절한 설명을 문서로 남길 수도 있다.



- 때론 람다가 메서드 참조보다 간결할 때가 있다.
  - 주로 메서드와 람다가 같은 클래스에 있을 때 그렇다.
  - `service.execute(GoshThisClassNameIsHumongous::action)`
  - `service.execute(() -> action());`
    - 메서드 참조 쪽은 더 짧지도, 더 명확하지도 않다.
      - 따라서 람다 쪽이 낫다.



- 메서드 참조의 유형은 다섯 가지로 가장 흔한 유형은 앞의 예에서 본 것처럼 **정적 메서드를 가리키는 메서드 참조**다.
  - 인스턴스 메서드를 참조하는 유형이 두 가지
    - 수신 객체(receiving object; 참조 대상 인스턴스)를 특정하는 **한정적(bound) 인스턴스 메서드 참조**
      - 한정적 참조는 근본적으로 정적 참조와 비슷하다.
      - 즉, 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같다.
    - 수신 객체를 특정하지 않는 **비한정적(unbound) 인스턴스 메서드 참조**
      - 비한정적 참조에서는 함수 객체를 적용하는 시점에 수신 객체를 알려준다.
      - 비한정적 참조는 주로 스트림 파이프라인에서의 매핑과 필터 함수에 쓰인다(아이템 45).
  - **클래스 생성자를 가리키는 메서드 참조**
  - **배열 생성자를 가리키는 메서드 참조**

| 메서드 참조 유형   | 예                     | 같은 기능을 하는 람다                                  |
| ------------------ | ---------------------- | ------------------------------------------------------ |
| 정적               | Integer::parsInt       | str → Integer.partInt(str)                             |
| 한정적(인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now();<br />t → then.isAfter(t) |
| 비한정적(인스턴스) | String::toLowerCase    | str → str.toLowerCase()                                |
| 클래스 생성자      | TreeMap<K,V>::new      | () → new TreeMap<K,V>()                                |
| 배열 생성자        | int[]::new             | len → new int[len]                                     |



## 아이템 44. 표준 함수형 인터페이스를 사용하라

> In summary, now that Java has lambdas, it is imperative that you design your APIs with lambdas in mind. Accept functional interface types on input and return them on output. It is generally best to use the standard interfaces provided in java.util.function.Function, but keep your eyes open for the relatively rare cases where you would be better off writing your own functional interface.

> 어제 자바도 람다를 지원한다. 여러분도 지금부터는 API를 설계할 때 람다도 염두에 두어야 한다는 뜻이다. 입력값과 반환값에 함수형 인터페이스 타입을 활용하라. 보통은 java.util.function 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋은 선택이다. 단, 흔치는 않지만 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을 수도 있음을 잊지 말자.



- 자바가 람다를 지원하면서 API를 작성하는 모범 사례도 크게 바뀌었다.
  - 예컨대 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴의 매력이 크게 줄었다.
  - 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다.
    - 이때 함수형 매개변수 타입을 올바르게 선택해야 한다.



- `java.util.function` 패키지를 보면 다양한 용도의 표준 함수형 인터페이스가 담겨있다.
  - 필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하자.
    - 그러면 API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워진다.
    - 또한 표준 함수형 인터페이스들은 유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호운용성도 크게 좋아질 것이다.
- `java.util.function` 패키지에는 총 43개의 인터페이스가 담겨 있다.
  - 전부 기억하긴 어렵겠지만, 기본 인터페이스 6개만 기억하면 나머지를 충분히 유추해 낼 수 있다.
- `Operator` 인터페이스는 인수가 1개인 `UnaryOperator`와 2개인 `BinaryOperator`로 나뉜다.
  - 반환값과 인수의 타입이 같은 함수
- `Predicate` 인터페이스는 인수 하나를 받아 `boolean`을 반환하는 함수
- `Function` 인터페이스는 인수와 반환 타입이 다른 함수
- `Supplier` 인터페이스는 인수를 받지 않고 값을 반환(혹은 제공)하는 함수
- `Consumer` 인터페이스는 인수를 하나 받고 반환값은 없는(특히 인수를 소비하는) 함수

| 인터페이스          | 함수 시그니처       | 예                  |
| ------------------- | ------------------- | ------------------- |
| `UnaryOperator<T>`  | T apply(T t)        | String::toLowerCase |
| `BinaryOperator<T>` | T apply(T t1, T t2) | BigInteger::add     |
| `Predicate<T>`      | boolean test(T t)   | Collection:isEmpty  |
| `Function<T,R>`     | R apply(T t)        | Arrays::asList      |
| `Supplier<T>`       | T get()             | Instant::now        |
| `Consumer<T>`       | void accept(T t)    | System.out::println |



- 예제에 `String::toLowerCase` 메서드는 파라미터가 없고 `BigInteger::add` 메서드는 파라미터가 하나?
  - "SANGCO"
    - `String` 인스턴스
    - `public interface UnaryOperator<T> extends Function<T, T> `
    - 타입 파라미터가 `<T, T>`
      - `T apply(T t);` 이렇게 되려나?

```java
public static void main(String[] args) {
    FuntionalInterface f = new FuntionalInterface();
    f.test(String::toLowerCase);
}

public void test(UnaryOperator<String> u) {
    u.apply("SANGCO");
  	// 이런식으로 사용하는 거니깐 이걸 인수 하나로 보는건가?
    // BinaryOperator도 이런식으로 생각하면 인수가 2개?
    "SANGCO".toLowerCase();
}
```



- 함수형 인터페이스에 있는 `static` 메서드와 디폴트 메서드
  - `static` 메서드는  `UnaryOperator.identity();` 이렇게 사용
  - 함수형 인터페이스 `Comparator`에 thenComparing, reversed 이런게 디폴트 메서드로 되어있음.



- 기본 인터페이스는 기본 타입인 `int`, `long`, `double`용으로 각 3개씩 변형이 생겨난다. 
  - 그 이름도 기본 인터페이스의 이름 앞에 해당 기본 타입 이름을 붙여 지었다.
    - `int`를 받는 `Predicate`는 `IntPredicate`
    - `long`을 받아 `long`을 반환하는 `BinaryOperator`는 `LongBinaryOperator`

- 이 변형들 중 유일하게 `Function`의 변형만 매개변수화됐다.
  - 정확히는 반환 타입만 매개변수화 됐다.
  - `LongFunction<int[]>`은 `long` 인수를 받아 `int[]`을 반환한다.
    - 원소의 타입이 `int[]`인 `LongFunction`을 뜻하는 매개변화수 타입
    - `Function<T, R>`, `LongFunction<R>`

```java
@FunctionalInterface 
public interface Function<T, R> {
    R apply(T t);
}
```

```java
@FunctionalInterface 
public interface LongFunction<R> {
    R apply(long value);
}
```



- `Function` 인터페이스에는 기본 타입을 반환하는 변형이 총 9개가 더 있다.
  - 인수와 같은 타입을 반환하는 함수는 `UnaryOperator`이므로, `Function` 인터페이스의 변형은 입력과 결과의 타입이 항상 다르다.
- 입력과 결과 타입이 모두 기본 타입이면 접두어로 `SrcToResult`를 사용한다.
  - 예컨대 `long`을 받아 `int`를 반환하면 `LongToIntFunction`이 되는 식이다(총 6개).
- 나머지는 입력이 객체 참조이고 결과가 `int`, `long`, `double`인 변형들로, 앞서와 달리 입력을 매개변수화하고 접두어로 `ToResult`를 사용한다.
  - `ToLongFunction<int[]>`은 `int[]` 인수를 받아 `long`을 반환한다(총 3개).



- 기본 함수형 인터페이스 중 3개에는 인수를 2개씩 받는 변형이 있다.
  - `BiPredicate<T, U>`
  - `BiFunction<T, U, R>`
    - 기본 타입을 반환하는 세 변형
      - `ToIntBiFunction<T, U, R>`
      - `ToLongBiFunction<T, U, R>`
      - `ToDoubleBiFunction<T, U, R>`
  - `BiConsumer<T, U>`

- `Consumer`에도 인수 2개(객체 참조와 기본 타입 하나) 받는 변형
  - `ObjDoubleConsumer<T>`
  - `ObjIntConsumer<T>`
  - `ObjLongConsumer<T>`
- 이렇게 해서 기본 인터페이스의 인수 2개짜리 변형은 총 9개다.
- `BooleanSupplier` 인터페이스는 `boolean`을 반환하도록 한 `Supplier`의 변형이다.
  - 이것이 표준 함수형 인터페이스 중 `boolean`을 이름에 명시한 유일한 인터페이스지만, `Predicate`와 그 변형 4개도 `boolean` 값을 반환할 수 있다.
- 이렇게 표준 함수형 인터페이스는 총 43개다.

- 다 외우기엔 수도 많고 규칙성도 부족하다.
  - 하지만 실무에서 자주 쓰이는 함수형 인터페이스 중 상당수를 제공하며, 필요할 때 찾아 쓸 수 있을 만큼은 범용적인 이름을 사용했다.



- 표주 함수형 인터페이스 대부분은 기본 타입만 지원한다.
  - 그렇다고 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자.
    - 동작은 하지만 "박싱된 기본 타입 대신 기본 타입을 사용하라"라는 아이템 61의 조언을 위배한다.
    - 특히 계산량이 많을 때는 성능이 처참히 느려질 수 있다.
- 이제 대부분 상황에서는 직접 작성하는 것보다 표준 함수형 인터페이스를 사용하는 편이 나음을 알았을 것이다.
  - 그렇다고 코드를 직접 작성해야 할 때는 언제인가?
    - 표준 인터페이스 중 필요한 용도에 맞는 게 없다면 직접 작성해야 한다.
- 구조적으로 똑같은 표준 함수형 인터페이스가 있더라도 직접 작성해야만 할 때가 있다.
  - `Comparator<T>` 인터페이스
    - 구조적으로는 `ToIntBiFunction<T, U>`와 동일하다.
    - 그런데 `Comparator`가 독자적인 인터페이스로 살아남아야 하는 이유가 몇 개 있다.
      - API에서 굉장히 자주 사용되는데, 지금의 이름이 그 용도를 아주 훌륭히 설명해준다.
      - 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있다.
      - 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 뜸뿍 담고 있다.
- 이상의 `Comparator` 특성을 정리하면 다음의 세 가지인데, 이 중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야 하는 건 아닌지 진중히 고민해야 한다.
  - 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
  - 반드시 따라야 하는 규약이 있다.
  - 유용한 디폴트 메서드를 제공할 수 있다.
- `@FunctionalInterface` 애너테이션을 사용하는 이유
  - 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
  - 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
  - 그 결과 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.
  - 그러니 직접 만든 함수형 인터페이스에는 항상 `@FunctionalInterface` 애너테이션을 사용하자.
- 함수형 인터페이스를 API에서 사용할 때의 주의점
  - 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안 된다.
  - ExecutorService의 submit 메서드는 `Callable<T>`를 받는 것과 `Runnable`을 받는 것을 다중정의했다.



## 아이템 45. 스트림은 주의해서 사용하라

> In summary, some tasks are best accomplished with streams, and others with iteration. Many tasks are best accomplished by combining the two approaches. There are no hard and fast rules for choosing which approach to use for a task, but there are some useful heuristics. In many cases, it will be clear which approach to use; in some cases, it won’t. **If you’re not sure whether a task is better served by streams or iteration, try both and see which works better.**

> 스트림을 사용해야 멋지게 처리할 수 있는 일이 있고, 반복 방식이 더 알맞은 일도 있다. 그리고 수많은 작업이 이 둘을 조합했을 때 가장 멋지게 해결된다. 어느 쪽을 선택하는 확고부동한 규칙은 없지만 참고할 만한 지침 정도는 있다. 어느 쪽이 나은지가 확연히 드러나는 경우가 많겠지만, 아니더라도 방법은 있다. **스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라.**



- 스트림 API는 다량의 데이터 처리 작업(순차적이든 병렬적이든)을 돕고자 자바 8에 추가되었다.
  - 이 API가 제공하는 추상 개념 중 핵심은 두 가지다.
    - 스트림(stream)은 데이터 원소의 유한 혹은 무한 시퀀스(sequence)를 뜻한다.
    - 스트림 파이프라인(stream pipeline)은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.
- 스트림의 원소들은 어디로부터든 올 수 있다.
  - 대표적으로는 컬렉션, 배열, 파일, 정규표현식 패턴 매처(matcher), 난수 생성기, 혹은 다른 스트림이 있다.
  - 스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값이다.
    - 기본 타입 값으로는 int, long, double 이렇게 세 가지를 지원한다.
- 스트림 파이프라인은 소스 스트림에서 시작해 종단 연산(terminal operation)으로 끝나며, 그 사이에 하나 이상의 중간 연산(intermediate operation)이 있을 수 있다.
  - 각 중간 연산은 스트림을 어떠한 방식으로 변환(transform)한다.
  - 종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가한다.
    - 원소를 정렬해 컬렉션에 담거나, 특정 원소 하나를 선택하거나, 모든 원소를 출력하는 식이다.
- 스트림 파이프라인은 지연 평가(lazy evaluation)된다. 
  - 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다.
  - 이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠다.
  - 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어인 no-op과 같으니, 종단 연산을 빼먹는 일이 절대 없도록 하자.
- 기본적으로 스트림 파이프라인은 순차적으로 수행된다.
  - 파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 parallel 메서드를 호출해주기만 하면 되나, 효과를 볼 수 있는 상황은 많지 않다(아이템 48).
- 스트림을 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어 진다.
  - 스트림을 언제 써야 하는지를 규정하는 확고부동한 규칙은 없지만, 참고 할 만한 노하우는 있다.



```java
// 코드 45-1 사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력한다.
public class IterativeAnagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word),
                        (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

```java
// 코드 45-2 스트림을 과하게 사용했다. - 따라 하지 말 것!
// 스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```

```java
// 코드 45-3 스트림을 적절히 활용하면 깔끔하고 명료해진다.
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```



- 람다 매개변수의 이름은 주의해서 정해야 한다.
  - 람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.



```java
// chars()가 반환하는 스트림의 원소는 char가 아닌 int 값이다.
"Hello world!".chars().forEach(System.out::print);

// 형변환을 명시적으로 해줘야 한다. 
// char 값들을 처리할 때는 스트림을 삼가는 편이 낫다.
"Hello world!".chars().forEach(x -> System.out.print((char) x));
```



- 스트림을 처음 쓰기 시작하면 모든 반복문을 스트림으로 바꾸고 싶은 유혹이 알겠지만, 서두르지 않는 게 좋다.
  - 스트림으로 바꾸는 게 가능할지라도 코드 가독성과 유지보수 측면에서는 손해르르 볼 수 있기 때문이다.
  - 중간 정도 복잡한 작업에도 스트림과 반복문을 적절히 조합하는 게 최선이다.
  - 그러니 기존 코드는 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아 보일 때만 반영하자.



- 함수 객체로는 할 수 없지만 코드 블록으로는 할 수 있는 일들이 있다.
  - 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다.
    - 하지만 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 건 불가능하다.
  - 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, break나 continue 문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있다.
    - 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다.
      - 하지만 람다로는 이 중 어떤 것도 할 수 없다.
- 다음 일들에는 스트림이 아주 안성맞춤이다.
  - 원소들의 시퀀스를 일관되게 변환한다.
  - 원소들의 시퀀스를 필터링한다.
  - 원소들의 시퀀스를 하나의 연산을 사용해 결합한다(더하기, 연결하기, 최솟값 구하기 등).
  - 원소들의 시퀀스를 컬렉션에 모은다(아마도 공통된 속성을 기준으로 묶어가며).
  - 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.
- 스트림으로 처리하기 어려운 일도 있다.
  - 대표적인 예로, 한 데이터가 파이프라인의 여러 단계(stage)를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기는 어려운 경우다.
  - 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃은 구조이기 때문이다.



- 스트림과 반복 중 어느 쪽을 써야 할지 바로 알기 어려운 작업도 많다.
  - 카드 덱을 초기화하는 작업을 생각해보자.
    - 카드는 숫자(rank)와 무늬(suit)를 묶은 불변 값 클래스이고, 숫자와 무늬는 모두 열거 타입이라 하자.
    - 이 작업은 두 집합의 원소들로 만들 수 있는 가능한 모든 조합을 계산하는 문제다.
      - 테카르트 곱

```java
// 코드 45-4 데카르트 곱 계산을 반복 방식으로 구현
private static List<Card> newDeck() {
  List<Card> result = new ArrayList<>();
  for (Suit suit : Suit.values())
    for (Rank rank : Rank.values())
      result.add(new Card(suit, rank));
  return result;
}
```

- 중간 연산으로 사용한 flatMap은 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합친다.
  - 이를 평탄화(flattening)라고도 한다.

```java
// 코드 45-5 데카르트 곱 계산을 스트림 방식으로 구현
private static List<Card> newDeck() {
  return Stream.of(Suit.values())
    .flatMap(suit ->
             Stream.of(Rank.values())
             .map(rank -> new Card(suit, rank)))
    .collect(toList());
}
```



- 이해하고 유지 보수하기에 처음 코드가 더 편한 프로그래머가 많겠지만, 두 번째인 스트림 방식을 편하게 생각하는 프로그래머도 있다.
  - 확신이 서지 않는 독자는 첫 번째 방식을 쓰는 게 더 안전할 것이다.
  - 스트림 방식이 나아 보이고 동료들도 스트림 코드를 이해할 수 있고 선호한다면 스트림 방식을 사용하자.



## 아이템 46. 스트림에서는 부작용 없는 함수를 사용하라

> In summary, the essence of programming stream pipelines is side-effect-free function objects. This applies to all of the many function objects passed to streams and related objects. The terminal operation forEach should only be used to report the result of a computation performed by a stream, not to perform the computa- tion. In order to use streams properly, you have to know about collectors. The most important collector factories are toList, toSet, toMap, groupingBy, and joining.

> 스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다. 스트림뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다. 종단 연산 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다. 계산 자체에는 이용하지 말자. 스트림을 올바로 사용하려면 수집기를 잘 알아둬야 한다. 가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining이다.



- 스트림 패러다임의 핵심은 계산을 일련의 변환(transformation)으로 재구성하는 부분이다.
  - 이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.
    - 순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다.
    - 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.
  - 이렇게 하려면 (중간 단계든 종단 단계든) 스트림 연산에 건네는 함수 객체는 모두 부작용(side effect)이 없어야 한다.

```java
// 코드 46-1 스트림 패러다임을 이해하지 못한 채 API만 사용했다 - 따라 하지 말 것!
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

- 이 코드의 모든 작업이 종단 연산인 forEach에서 일어나는데, 이때 외부 상태(빈도표)를 수정하는 람다를 실행하면서 문제가 생긴다.

```java
// 코드 46-2 스트림을 제대로 활용해 빈도표를 초기화한다.
Map<String, Long> freq;
// tokens를 사용해 스트림을 얻었는데 자바 9부터 지원.
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words
            .collect(groupingBy(String::toLowerCase, counting()));
}
```

- 앞서와 같은 일을 하지만, 이번엔 스트림 API를 제대로 사용했다.
  - 그뿐만 아니라 짧고 명확하다.
- forEach 연산은 종단 연산 중 기능이 가장 적고 가장 '덜' 스트림답다.
  - 대놓고 반복적이라서 병렬화할 수도 없다.
- forEach 연산으 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자.
  - 물론 가끔은 스트림 계산 결과를 기존 컬렉션에 추가하는 등의 다른 용도로도 쓸 수 있다.



- Stream 클래스
  - `<R, A> R collect(Collector<? super T, A, R> collector);`
    - 인수로 `Collector` 타입을 받는다.
    - `Collector` 타입을 반환하는 `java.util.stream.Collectors` 클래스의 메서드들



- java.util.stream.Collectors 클래스는 메서드를 무려 39개나 가지고 있다.
  - 축소(reduction) 전략을 캡슐화한 블랙박스 객체라고 생각하기 바란다.
  - 여기서 축소는 스트림의 원소들을 객체 하나에 취합한다는 뜻이다.
  - 수집기가 생성하는 객체는 일반적으로 컬렉션이며, 그래서 "collector"라는 이름을 쓴다.
- 수집기를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다.
  - 수집기는 총 세 가지로, toList(), toSet(), toCollection(collectionFactory)이다.

```java
// 코드 46-3 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```

- 마지막 toList는 Collectors의 메서드다.
  - 이처럼 Collectors의 멤버를 정적 임포트하여 쓰면 스트림 파이프라인 가독성이 좋아져, 흔히들 이렇게 사용한다.
- comparing 메서드는 키 추출 함수를 받는 비교자 생성 메서드(아이템 14)다.
  - 그리고 한정적 메서드 참조이자, 여기서 키 추출 함수로 쓰인 freq::get은 입력받은 단어(키)를 빈도표에서 찾아(추출) 그 빈도를 반환한다.



- Collectors의 나머지 36개 메서드들도 알아보자.



```java
// 코드 46-4 toMap 수집기를 사용하여 문자열을 열거 타입 상수에 매핑한다.
private static final Map<String, Operation> stringToEnum =
  	Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

```java
public static <T, K, U> Collector<T, ?, Map<K,U>> toMap(
  	  Function<? super T, ? extends K> keyMapper,
  	  Function<? super T, ? extends U> valueMapper) {
  
    return toMap(keyMapper, valueMapper, throwingMerger(), HashMap::new);
  
}
```

- 가장 간단한 맵 수집기는 toMap(keyMapper, valueMapper)로, 보다시피 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.



```java
// 코드 46-5 각 키와 해당 키의 특정 원소를 연관 짓는 맵을 생성하는 수집기
Map<Artist, Album> toopHits = albums.collect(
  	toMap(Album::artist, a -> a), maxBy(comparing(Album::sales)))
```

```java
public static <T, K, U> Collector<T, ?, Map<K,U>> toMap(
  		Function<? super T, ? extends K> keyMapper,
			Function<? super T, ? extends U> valueMapper,
  	  BinaryOperator<U> mergeFunction) {
        
    return toMap(keyMapper, valueMapper, mergeFunction, HashMap::new);
  
}
```

- 인수 3개를 받는 toMap은 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다.
- 여기서 비교자로는 BinaryOperator에서 정적 임포트한 maxBy라는 정적 팩터리 메서드를 사용했다.
  - maxBy는 Comparator<T>를 입력받아 BinaryOperator<T>를 돌려준다.
- 말로 풀어보자면 "앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다."
- 인수가 3개인 toMap은 충돌이 나면 마지막 값을 취하는(last-write-wins) 수집기를 만들 때도 유용하다.
  - 많은 스트림의 결과가 비결정적이다.
  - 하지만 매핑 함수가 키 하나에 연결해준 값들이 모두 같을 때, 혹은 값이 다르더라도 모두 허용되는 값일 때 이렇게 동작하는 수집기가 필요하다.



```java
// 코드 46-7 마지막에 쓴 값을 취하기 수집기
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

```java
public static <T, K, U, M extends Map<K, U>> Collector<T, ?, M> toMap(
      Function<? super T, ? extends K> keyMapper,
      Function<? super T, ? extends U> valueMapper,
      BinaryOperator<U> mergeFunction,
      Supplier<M> mapSupplier) {
  
  	BiConsumer<M, T> accumulator = (map, element) -> map.merge(keyMapper.apply(element),
                                              			 valueMapper.apply(element), mergeFunction);
    return new CollectorImpl<>(mapSupplier, accumulator, mapMerger(mergeFunction), CH_ID);
  
}
```

- 세 번째이자 마지막 toMap은 네 번째 인수로 맵 팩터리를 받는다.
  - 이 인수로는 EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직접 지정할 수 있다.



- 이상의 세 가지 toMap에는 변종이 있다.
  - 그중 toConcurrentMap은 병렬 실행된 후 결과로 ConcurrentHahMap 인스턴스를 생성한다.



- Collectors가 제공하는 또 다른 메서드인 groupingBy를 알아보자.
  - 이 메서드는 입력으로 분류 함수(classifier)를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다.
  - 다중정의된 groupingBy 중 형태가 가장 간단한 것은 분류 함수 하나 인수로 받아 맵을 반환한다.
    - 반환된 맵에 담긴 각각의 값은 해당 카테고리에 속하는 원소들을 모두 담은 리스트다.



```java
words.collect(groupingBy(word -> alphabetize(word)))
```

- groupingBy가 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와 함께 다운스트림(downstream) 수집기도 명시해야 한다.
  - 다운스트림 수집기의 역할은 해당 카테코그의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다.
  - 이 매개변수를 사용하는 가장 간단한 방법은 toSet()을 넘기는 것이다.
    - 그러면 groupingBy는 원소들의 리스트가 아닌 집합(Set)을 값으로 갖는 맵을 만들어낸다.
- toSet() 대신 toCollection(collectionFactory)를 건네는 방법도 있다.
  - 예상 할 수 있듯이 이렇게 하면 리스트나 집합 대신 컬렉션을 값으로 갖는 맵을 생성한다.
  - 원하는 컬렉션 타입을 선택할 수 있다는 유연성은 덤이다.

```java
Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase, counting()));
```

- 다운스트림 수집기로 countin()을 건네는 방법도 있다.
  - 이렇게 하면 각 카테고리(키)를 (원소를 담은 컬렉션이 아닌) 해당 카테고리에 속하는 원소의 개수(값)와 매핑한 맵을 얻는다.



```java
Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier, 
                              Supplier<M> mapFactory,
															Collector<? super T, A, D> downstream) { ... }
```

- groupingBy의 세 번째 버전은 다운스트림 수집기에 더해 맵 팩터리도 지정할 수 있게 해준다.
  - mapFactory 매개변수가 downStream 매개변수보다 앞에 놓인다.
  - 이 버전의 groupingBy를 사용하면 맵과 그 안에 담긴 컬렉션의 타입을 모두 지정할 수 있다.
    - 예컨대 값이 TreeSet인 TreeMap을 반환하는 수집기를 만들 수 있다.



- Collectors에 정의되어 있지만 '수집'과는 관련이 없다.
  - 그중 minBy와 maxBy는 인수로 받은 비교자를 이용해 스트림에서 값이 가장작은 혹은 가장 큰 원소를 찾아 반환한다.

- Collectors의 마지막 메서드는 joining이다.
  - 이 메서드는 (문자열 등의) CharSequence 인스턴스의 스트림에만 적용할 수 있다.
  - 이 중 매개변수가 없는 joining은 단순히 원소들을 연결(concatenate)하는 수집기를 반환한다.
  - 한편 인수 하나짜리 joining은 CharSequence 타입의 구분문자(delimiter)를 매개변수로 받는다.
    - 연결 부위에 이 구분문자를 삽입하는데, 예컨대 구분문자로 쉼표(,)를 입력하면 CSV 형태의 문자열을 만들어준다.
  - 인수 3개짜리 joining은 구분문자에 더해 접두문자(prefix)와 접미문자(suffix)도 받는다.



## 아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

> In summary, when writing a method that returns a sequence of elements, remember that some of your users may want to process them as a stream while others may want to iterate over them. Try to accommodate both groups. If it’s fea- sible to return a collection, do so. If you already have the elements in a collection or the number of elements in the sequence is small enough to justify creating a new one, return a standard collection such as ArrayList. Otherwise, consider implementing a custom collection as we did for the power set. If it isn’t feasible to return a collection, return a stream or iterable, whichever seems more natural. If, in a future Java release, the Stream interface declaration is modified to extend Iterable, then you should feel free to return streams because they will allow for both stream processing and iteration.

> 원소 시퀀스를 반환하는 메서드를 작성할 때는, 이를 스트림으로 처리하기를 원하는 사용자와 반복으로 처리하길 원하는 사용자가 모두 있을 수 있음을 떠올리고, 양쪽을 다만족시키려 노력하자. 컬렉션을 반환할 수 있다면 그렇게 하라. 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList 같은 표준 컬렉션에 담아 반환하라. 그렇지 않으면 앞서의 멱집합 예처럼 전용 컬렉션을 구현할지 고민하라. 컬렉션을 반환하는 게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하라. 만약 나중에 Stream 인터페이스가 Iterable 중 더 자연스러운 것을 반환하라. 만약 나중에 Stream 인터페이스가 Iterable을 지원하도록 자바가 수정된다면, 그때는 안심하고 스트림을 반환하면 될 것이다(스트림 처리와 반복 모두에 사용할 수 있으니).



- 원소 시퀀스, 즉 일련의 원소를 반환하는 메서드는 수없이 많다.
  - 자바 7까지는 이런 메서드의 반환 타입으로 Collection, Set, List 같은 컬렉션 인터페이스, 혹은 Iterable이나 배열을 썼다.
  - 기본은 컬렉션 인터페이스다.
- for-each 문에서만 쓰이거나 반환된 원소 시퀀스가 (주로 contains(Object) 같은) 일부 Collection 메서드를 구현할 수 없을 때는 Iterable 인터페이스를 썼다.
- 반환 원소들이 기본 타입이거나 성능에 민감한 상황이라면 배열을 썼다.
- 자바 8이 스트림이라는 개념을 들고 오면서 이 선택이 아주 복잡한 일이 되어버렸다.
- 아이템 45에서 이야기 했듯이 스트림은 반복(iteration)을 지원하지 않는다.
  - 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다.
- 사실 Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함할 뿐만 아니라, Iterable 인터페이스가 정의한 방식대로 동작한다.
  - 그럼에도 for-each로 스트림을 반복할 수 없는 까닭은 바로 Stream이 Iterable을 확장(extend)하지 않아서다.



- 작동은 하지만 실전에 쓰기에는 너무 난잡하고 직관성이 떨어진다.
  - 다행히 어댑터 메서드를 사용하면 상황이 나아진다.
  - 자바는 이런 메서드를 제공하지 않지만 다음 코드와 같이 쉽게 만들어낼 수 있다.

```java
// 코드 47-1 자바 타입 추론의 한계로 컴파일되지 않는다.
// 이 오류를 바로잡으려면 메서드 참조를 매개변수화된 Iterable로 적절히 형변환해줘야 한다.
for (ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
  	// 프로세스를 차단한다.
} 
```

```java
// 코드 47-2 스트림을 반복하기 위한 '끔찍한' 우회 방법
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
  	// 프로세스를 차단한다.
} 
```



- 객체 시퀀스를 반환하는 메서드를 작성하는데, 이 메서드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 마음 놓고 스트림을 반환하게 해주자.
  - 반대로 반환된 객체들이 반복문에서만 쓰일 걸 안다면 Iterable을 반환하자.
- 공개 API를 작성할 때는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰려는 사람 모두를 배려해야 한다.

```java
// 코드 47-3 Stream<E>를 Iterable<E>로 중개해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator;
}
```

```java
// 코드 47-4 Iterable<E>를 Stream<E>로 중개해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false);
}
```



- Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다.
  - 따라서 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.
  - Arrays 역시 Arrays.asList와 Stream.of 메서드로 손쉽게 반복과 스트림을 지원할 수 있다.
- 반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList나 HashSet 같은 표준 컬렉션 구현체를 반환하는 게 최선일 수 있다.
  - 하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다.



- 반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 검토해보자.
- 주어진 집합의 멱집합(한 집합의 모든 부분지합을 원소로 하는 집합)을 반환하는 상황
  - {a, b, c}의 멱집합은 {{}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c}}
  - 멱집합을 표준 컬렉션 구현체에 저장하려는 생각은 위험하다.
  - 하지만 AbstractList를 이용하면 훌륭한 전용 컬렉션을 손쉽게 구현할 수 있다.
- 비결은 멱집합을 구성하는 각 원소의 인덱스를 비트 백터로 사용하는 것이다.
  - 인덱스의 n번째 비트 값은 멱집합의 해당 원소가 원래 집합의 n번째 원소를 포함하는지 여부를 알려준다.
    - 따라서 0부터 2^n - 1까지의 이진수와 원소 n개인 집합의 멱집합과 자연스럽게 매핑된다.

```java
public class PowerSet {
    // 코드 47-5 입력 집합의 멱집합을 전용 컬렉션에 담아 반환한다.
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException(
                "집합에 원소가 너무 많습니다(최대 30개).: " + s);
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                // 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱 것과 같다.
                return 1 << src.size();
            }

            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }
}
```



- (반복이 시작되기 전에는 시퀀스의 내용을 확정할 수 없는 등의 사유로) contains와 size를 구현하는 게 불가능할 때는 컬렉션보다는 스트림이나 Iterable을 반환하는 편이 낫다.

```java
// 코드 47-6 입력 리스트의 모든 부분리스트를 스트림으로 반환한다.
public static <E> Stream<List<E>> of(List<E> list) {
  return Stream.concat(Stream.of(Collections.emptyList()),
                       prefixes(list).flatMap(SubLists::suffixes));
}

private static <E> Stream<List<E>> prefixes(List<E> list) {
  return IntStream.rangeClosed(1, list.size())
    .mapToObj(end -> list.subList(0, end));
}

private static <E> Stream<List<E>> suffixes(List<E> list) {
  return IntStream.range(0, list.size())
    .mapToObj(start -> list.subList(start, list.size()));
}
```

```java
// 코드 47-7 입력 리스트의 모든 부분리스트를 스트림으로 반환한다(빈 리스트는 제외).
public static <E> Stream<List<E>> of(List<E> list) {
    return IntStream.range(0, list.size())
            .mapToObj(start ->
                    IntStream.rangeClosed(start + 1, list.size())
                            .mapToObj(end -> list.subList(start, end)))
            .flatMap(x -> x);
}
```



- 이상으로 스트림을 반환하는 두 가지 구현을 알아봤는데, 모두 쓸만은 하다.
  - 하지만 반복을 사용하는 게 더 자연스러운 상황에서도 사용자는 그냥 스트림을 쓰거나 Stream을 Iterable로 변환해주는 어댑터를 이용해야 한다.
  - 하지만 이러한 어댑터는 클라이언트 코드를 어수선하게 만들고 (내 컴퓨터에서는) 2.3배가 더 느리다.
  - (책에 싣지 않은) 직접 구현한 전용 Collection을 사용하니 코드는 훨씬 지저분해졌지만, 스트림을 활용한 구현보다 (내 컴퓨터에서) 1.4배 빨랐다.



## 아이템 48. 스트림 병렬화는 주의해서 적용하라

> In summary, do not even attempt to parallelize a stream pipeline unless you have good reason to believe that it will preserve the correctness of the computation and increase its speed. The cost of inappropriately parallelizing a stream can be a program failure or performance disaster. If you believe that parallelism may be justified, ensure that your code remains correct when run in parallel, and do careful performance measurements under realistic conditions. If your code remains correct and these experiments bear out your suspicion of increased performance, then and only then parallelize the stream in production code.

> 계산도 올바로 수행하고 성능도 빨라질 거라는 확신 없이는 스트림 파이프라인 병렬화는 시도조차 하지 말자. 스트림을 잘못 병렬화하면 프로그램을 오동작하게 하거나 성능을 급격히 떨어뜨린다. 병렬화하는 편이 낫다고 믿더라도, 수정 후의 코드가 여전히 정확한지 확인하고 운영 환경과 유사한 조건에서 수행해보면 성능지표를 유심히 관찰하라. 그래서 계산도 정확하고 성능도 좋아졌음이 확실해졌을 때, 오직 그럴 때만 병렬화 버전 코드를 운영 코드에 반영하라.



- 주류 언어 중, 동시성 프로그래밍 측면에서 자바는 항상 앞서갔다.
  - 자바 8부터는 parallel 메서드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 스트림을 지원했다.
  - 이처럼 자바로 동시성 프로그램을 작성하기가 점점 쉬워지고는 있지만, 이를 올바르고 빠르게 작성하는 일은 어전히 어려운 작업이다.
  - 동시성 프로그래밍을 할 때는 안전성(safety)과 응답 가능(liveness) 상태를 유지하기 위해 애써야 하는데, 병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없다.



```java
// 병렬 스트림을 사용해 처음 20개의 메르센 소수를 생성하는 프로그램 (291쪽 코드 48-1의 병렬화 버전)
// 주의: 병렬화의 영향으로 프로그램이 종료하지 않는다.
public class ParallelMersennePrimes {
    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .parallel() // 스트림 병렬화
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }

    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}
```

- 환경이 아무리 좋더라도 데이터 소스가 Stream.iterate거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.
- 대체로 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 배열, long 범위일 때 병렬화의 효과가 가장 좋다.
  - 이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 스레드에 분배하기에 좋다는 특징이 있다.
- 이 자료구조들의 또 다른 중요한 공통점은 원소들을 순차적으로 실행할 때의 참조 지역성(locality of reference)이 뛰어나다는 것이다.
  - 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻이다.
  - 참조들이 가리키는 실제 객체가 메모리에서 서로 떨어져 있을 수 있는데, 그러면 참조 지역성이 나빠진다.
    - 참조 지역성이 낮으면 스레드는 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리며 대부분 시간을 멍하니 보내게 된다. 
- 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 아주 중요한 요소로 작용한다.
  - 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다.
  - 기본 타입 배열에서는 (참조가 아닌) 데이터 자체가 메모리에 연속해서 저장되기 때문이다.
- 스트림 파이프라인의 종단 연산의 동작 방식 역시 병렬 수행 효율에 영향을 준다.
  - 종단 연산 중 병렬화에 가장 적합한 것은 축소(reduction)다.
    - 축소는 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업으로, Stream의 reduce 메서드 중 하나, 혹은 min, max, count, sum 같이 완성된 형태로 제공되는 메서드 둥 하나를 선택해 수행한다.
  - anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환되는 메서드도 병렬화에 적합하다.
  - 반면, 가변 축소(mutable reduction)를 수행하는 Stream의 collect 메서드는 병렬화에 적합하지 않다.
    - 컬렉션들을 합치는 부담이 크기 때문이다.

- 직접 구현한 Stream, Iterable, Collection이 병렬화의 이점을 제대로 누리게 하고 싶다면 spliterator 메서드를 반드시 재정의하고 결과 스트림의 병렬화 성능을 강도 높게 테스트하라.
- 스트림을 잘못 병렬화하면 (응답 불가를 포함해) 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다.
- 스트림 병렬화는 오직 성능 최적화 수단임을 기억해야 한다.
  - 다른 최적화와 마찬가지로 변경 전후로 반드시 성능을 테스트하여 병렬화를 사용할 가치가 있는지 확인해야 한다(아이템 67).
  - 이상적으로는 운영 시스템과 흡사한 환경에서 테스트하는 것이 좋다.
- 앞으로 여러분이 스트림 파이프라인을 병렬화할 일이 적어질 것처럼 느껴졌다면, 그건 진짜 그렇기 때문이다.

- 조건이 잘 갖춰지면 parallel 메서드 호출 하나도 거의 프로세서 코어 수에 비례하는 성능 향상을 만끽할 수 있다.
  - 머신러닝과 데이터 처리 같은 특정 분야에서는 이 성능 개선의 유혹을 뿌리치기 어려울 것이다.

```java
public class ParallelPrimeCounting {
    // 코드 48-3 소수 계산 스트림 파이프라인 - 병렬화 버전
    static long pi(long n) {
        return LongStream.rangeClosed(2, n)
                .parallel()
                .mapToObj(BigInteger::valueOf)
                .filter(i -> i.isProbablePrime(50))
                .count();
    }

    public static void main(String[] args) {
        System.out.println(pi(10_000_000));
    }
}
```















