

## 아이템 49. 매개변수가 유효한지 검사하라

> To summarize, each time you write a method or constructor, you should think about what restrictions exist on its parameters. You should document these restrictions and enforce them with explicit checks at the beginning of the method body. It is important to get into the habit of doing this. The modest work that it entails will be paid back with interest the first time a validity check fails.

> 메서드나 생성자를 작성할 때면 그 매개변수들에 어떤 제약이 있을지 생각해야 한다. 그 제약들을 문서화하고 메서드 코드 시작 부분에서 명시적으로 검사해야 한다. 이런 습관을 반드시 기르도록 하자. 그 노력은 유효성 검사가 실제 오류를 처음 걸러낼 때 충분히 보상받을 것이다.



- 매개변수의 제약은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 한다.
  - "오류는 가능한 한 빨리 (발생한 곳에서) 잡아야 한다"는 일반 원칙의 한 사례이기도 하다.
    - 오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워지고, 감지하더라도 오류의 발생 지점을 찾기 어려워진다.



#### 매개변수 검사

- 메서드 몸체가 실행되기 전에 매개변수를 확인한다면 잘못된 값이 넘어왔을 때 즉각적이고 깔끔한 방식으로 예외를 던질 수 있다.
-  매개변수 검사를 제대로 하지 못하면 생기는 문제
  - 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.
    - 더 나쁜 상황
      - 메서드가 잘 수행되지만 잘못된 결과를 반환하는 상황
    - 한층 더 나쁜 상황
      - 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들고 미래의 알 수 없는 시점에 이 메서드와는 관련 없는 오류를 내는 상황
- 매개변수 검사에 실패하면 실패 원자성(failure atomicity, 아이템 76)을 어기는 결과를 낳을 수 있다.





어떤 제약



문서화



명시적 검사 







## 아이템 50. 적시에 방어적 복사본을 만들어라

> In summary, if a class has mutable components that it gets from or returns to its clients, the class must defensively copy these components. If the cost of the copy would be prohibitive *and* the class trusts its clients not to modify the compo- nents inappropriately, then the defensive copy may be replaced by documentation outlining the client’s responsibility not to modify the affected components.





## 아이템 51. 메서드 시그니처를 신중히 설계하라



## 아이템 52. 다중정의는 신중히 사용하라

> To summarize, just because you can overload methods doesn’t mean you should. It is generally best to refrain from overloading methods with multiple signatures that have the same number of parameters. In some cases, especially where constructors are involved, it may be impossible to follow this advice. In these cases, you should at least avoid situations where the same set of parameters can be passed to different overloadings by the addition of casts. If this cannot be avoided, for example, because you are retrofitting an existing class to implement a new interface, you should ensure that all overloadings behave identically when passed the same parameters. If you fail to do this, programmers will be hard pressed to make effective use of the overloaded method or constructor, and they won’t understand why it doesn’t work.





## 아이템 53. 가변인수는 신중히 사용하라

> In summary, varargs are invaluable when you need to define methods with a variable number of arguments. Precede the varargs parameter with any required parameters, and be aware of the performance consequences of using varargs.





## 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

> In summary, **never return** **null** **in place of an empty array or collection.** It makes your API more difficult to use and more prone to error, and it has no performance advantages.



- null을 반환하면 메서드를 사용하는 클라이언트에서 방어 코드를 넣어줘야 한다.
  - 방어 코드가 없다면 수년 뒤에 뜸금없이 에어가 날 수도..  

- 빈 컨테이너를 할당하는데 비용이 많이 드니 null을 반환하는게 낫다?

  - 임마가 성능 저하의 주범이 아닐 가능성이 높다.

  - 빈 컬렉션과 배열을 굳이 새로 할당하지 않고도 반환할 수 있다.

    - <T> T[] List.toArray(T[] a)
      - a가 충분히 크면 a에 담아서 반환, 그렇지 않으면 새로 배열 만들고 담아서 반환 


```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheeseInStock);
}

// TODO 최적화에 해당 하니 꼭 필요할 때만 써라? 그냥 쓰면 안되나?
// TODO cheeseInStock을 반환하지 않고 이번에도 new ArrayList<>(cheeseInStock)을 반환? 왜?
public List<Cheese> getCheeses() {
    return cheeseInStock.isEmpty() 
      // 매번 똑같은 빈 '불변' 컬렉션을 반환
      ? Collections.emptyList(): new ArrayList<>(cheeseInStock);
  // Collections.emptySet, Collections.emptyMap
}

public Cheese[] getCheeses() {
    return cheeseInStock.toArray(new Cheese[0]);
}

private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
  // return cheeseInStock.toArray(new Cheeses[cheeseInStock.size()]);
  // TODO 왜 성능이 떨어지는 걸까? 크기가 충분하지 않을 때 새로 생성되는 일이 없을텐데
}
```



## 아이템 55. 옵셔널 반환은 신중히 하라 


> In summary, if you find yourself writing a method that can’t always return a value and you believe it is important that users of the method consider this possibility every time they call it, then you should probably return an optional. You should, however, be aware that there are real performance consequences associated with returning optionals; for performance-critical methods, it may be better to return a null or throw an exception. Finally, you should rarely use an optional in any other capacity than as a return value.



#### 자바 8 이전

- 자바 8 이전에는 메서드가 값을 반환할 수 없을 때
  - 예외 던지기
    - 허점
      - 예외는 진짜 예외적인 상황에
      - 예외 생성 시 스택 추적 전체를 캡쳐하는 비용이 상당함.
  - null (반환 타입이 객체 참조라면)
    - 허점
      - 메소드를 사용하는 측에서 별도의 null 처리 코드를 추가해야한다.



#### 자바 8 이후

- 옵셔널을 반환
  - 예외를 던지는 것 보다는 유연하고 사용하기 쉽다.
  - null을 반환하는 것 보다는 오류 가능성이 적다.
- Optional.of(value)에 null을 넣으면 NullPointerException을 던지니 주의
  - null 값도 허용하는 옵셔널을 만들려면 Optional.ofNullable(value)를 사용 

```java
    // 코드 55-2 컬렉션에서 최댓값을 구해 Optional<E>로 반환한다. (327쪽)
    public static <E extends Comparable<E>>
    Optional<E> max(Collection<E> c) {
        if (c.isEmpty())
            return Optional.empty();

        E result = null;
        for (E e : c)
            if (result == null || e.compareTo(result) > 0)
                result = Objects.requireNonNull(e);

				// 위에서 isEmpty()로 걸렀으니 null 값이 안들어 가겠지?
        return Optional.of(result);
    }
```



- 스트림의 종단 연산 중 상당수가 옵셔널을 반환한다.

```java
 // 코드 55-3 컬렉션에서 최댓값을 구해 Optional<E>로 반환한다. - 스트림 버전 (328쪽)
    public static <E extends Comparable<E>>
    Optional<E> max(Collection<E> c) {
        return c.stream().max(Comparator.naturalOrder());
    }
```



#### 옵셔널 활용

- null 이나 예외 대신 옵셔널 반환을 선택해야 하는 기준은?
  - 옵셔널은 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려 준다.
  - 메서드가 옵셔널을 반환하면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야 한다.
    - 기본값을 설정하는 방법
      - .orElse()
      - .orElseThrow()
      - .get()
      - .orElseGet()
    - 더 특별한 쓰임에 대비한 메서드
      - filter, map, flatMap, ifPresent
      - 자바 9에서는 Optional에 stream() 메서드가 추가 되었다.



#### 옵셔널 주의사항

- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다.
- 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T>를 반환 하는걸 디폴트로 가자.
- 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.
  - OptionalInt, OptionalLong, OptionalDouble 등 을 쓰자.
- 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는게 적절한 상황은 거의 없다. 
  - 옵셔널을 인스턴스 필드에 저장해두는 상황
    - 인스턴스의 필드 중 필수가 아닌 필드가 있고 그 타입이 기본 타입이라면 값이 없음을 나타낼 방법이 마땅치 않다.
      - 이 경우 필드 자체를 옵셔널로 만들어 주는 것도 좋은 방법이다.



## 아이템 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라

> To summarize, documentation comments are the best, most effective way to document your API. Their use should be considered mandatory for all exported API elements. Adopt a consistent style that adheres to standard conventions. Remember that arbitrary HTML is permissible in documentation comments and that HTML metacharacters must be escaped.

