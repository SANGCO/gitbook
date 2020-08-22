

## 아이템 26. 로 타입은 사용하지 말라

> In summary, using raw types can lead to exceptions at runtime, so don’t use them. They are provided only for compatibility and interoperability with legacy code that predates the introduction of generics. As a quick review, Set<Object> is a parameterized type representing a set that can contain objects of any type, Set<?> is a wildcard type representing a set that can contain only objects of some unknown type, and Set is a raw type, which opts out of the generic type system. The first two are safe, and the last is not.

> 로 타입을 사용하면 런타임에 예외가 일어날 수 있으니 사용하면 안 된다. 로 타입은 제네릭이 도입되기 이전 코드와의 호환성을 위해 제공될 뿐이다. 빠르게 훑어보자면, Set<Object>는 어떤 타입의 객체도 저장할 수 있는 매개변수화 타입이고, Set<?>는 모종의 타입 객체만 저장할 수 있는 와일드카드 타입이다. 그리고 이들의 로 타입인 Set은 제네릭 타입 시스템에 속하지 않는다. Set<Object>와 Set<? >는 안전하지만, 로 타입인 Set은 안전하지 않다.



### 용어 정리

- 아따 많다. 타입 파라미터와 타입들을 분류해보자.

| 한글 용어                    | 영문                           | 예                                |
| :--------------------------- | ------------------------------ | --------------------------------- |
| 실제 타입 파리미터(매개변수) | actual type parameter          | <String>                          |
| (정규) 타입 파라미터         | formal type parameter          | <E>                               |
| 한정적 타입 파라미터         | bounded type parameter         | <E extends Number>                |
| 재귀적 타입 한정             | recursive type bound           | <T extends Comparable<T>>         |
| 이거는 부르는 이름이 없나?   | recursive wildcard type bound? | <T extends Comparable<? super T>> |
|                              |                                |                                   |
| 로 타입                      | raw type                       | List                              |
| 매개변수화 타입              | parameterized type             | List<String>                      |
| 제네릭 타입                  | generic type                   | List<E>                           |
| 비한정적 와일드카드 타입     | unbounded wildcard type        | List<?>                           |
| 한정적 와일드카드 타입       | bounded wildcard type          | List<? extends Number>            |
|                              |                                |                                   |
| 제네릭 메서드                | generic method                 | static <E> List<E> asList(E[] a)  |
| 타입 토큰                    | type token                     | String.class                      |



### 로타입

- 이 책 전반에서 줄기차게 이야기하듯, 오류는 가능한 한 발생 즉시, 이상적으로는 컴파일할 때 발견하는 것이 좋다.
  - 밑에 예제는 로 타입을 쓴 절대 따라 하면 안되는 예제
    - 컴파일, 실행은 되지만 런타임에 문제가 발생하고 문제 발생 지점과 원인을 제공한 코드가 멀리 떨어져 있을 가능성이 있어 원인 찾으려면 전체를 훑어봐야 할 수도 있다. 
  - 매개변수화된 컬렉션 타입으로 만들어 주면 엉뚱한 타입을 넣으려 하면 컴파일 오류가 발생한다.
    - `incompatible types: Coin connot be converted`

```java
// 코드 26-1 컬렉션의 로 타입 - 따라 하지 말 것!
// Stamp 인스턴스만 취급한다.
private final Collection stamps = ...;
// 실수로 동전을 넣는다.
stamps.add(new Coin(...)); // "unchecked cll" 경고를 내빝기는 하지만 컴파일이 된다.
```

```java
// 코드 26-2 반복자의 로 타입 - 따라 하지 말 것!
for (Iterator i = stamps.iterator(); i.hasNext();) {
    Stamp stamp = (Stamp) i.next(); // ClassCastException 발생
    stamp.cancel();
}
```

```java
// 코드 26-3 매개변수화된 컬렌션 타입 - 타입 안정성 확보!
private final Collection<Stamp> stamps = ...;
```



- **로 타입**을 쓰는 걸 언어 차원에서 막아 놓지는 않았지만 **절대로 써서는 안된다.**
  - 로 타입을 쓰면 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 된다.
  - 절대 써서는 안 되는 로 타입을 만든 이유는 제네릭이 도입되기 전 코드와의 호환성 때문이다.
    - 이 마이그레이션 호환성을 위해 로 타입을 지원하고 제네릭 구현에는 소거(erasure; 아이템 28) 방식을 사용하기로 했다.



### 로 타입과 매개변수화 타입

- 로 타입 `List`와 매개변수화 티입인 `List<Object>`의 차이는 무엇일까나?
  - 매개변수로 `List`를 받는 메서드 `List<String>`을 넘길 수 있지만, `List<Object>`를 받는 메서드에는 넘길 수 없다.
  - `List<String>`은 로 타입인 `List`의 하위 타입이지만, `List<Object>`의 하위 타입은 아니다(아이템 28).
  - `List<Object>` 같은 매개변수화 타입을 사용할 때와 달리 `List` 같은 로 타입을 사용하면 타입 안전성을 잃게 된다.

```java
// 코드 26-4 런타임에 실패한다. - unsafeAdd 메서드가 로 타입(List)을 사용
public class Raw {
    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다.
    }

		// List list를 List<Object> list로 변경하면 컴파일조차 되지 않는다. 
    // 오류를 실행전에 컴파일할 때 발견!   
    private static void unsafeAdd(List list, Object o) {
        list.add(o);
    }
}
```



### 비한정적 와일드카드 타입

- 제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않다면 물음표(?) 와일드카드를 사용하자.
  - 제네릭 타입인 `Set<E>`의 비한정적 와일드카드 타입은 `Set<?>`이다.
    - 이것이 어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 `Set` 타입이다. 
  - 와일드카드 타입은 안전하고, 로 타입은 안전하지 않다.
    - 로 타입 컬렉션에는 아무 원소나 넣을 수 있으니 타입 불변식을 훼손하기 쉽다.
    - 반면, `Collection<?>`에는 (`null` 외에는) 어떤 원소도 넣을 수 없다.
      - `String`이든 뭐든 넣으려고 하면 컴파일 오류 발생
    - 비한정적 와일드카드 타입으로 하면 `Set<String>`, `Set<Integer>` 어떤 타입이든지 다 파라미터로 던질 수 있네.
      - 근데 파라미터로 넘기는 컬렉션에는 null 빼고는 아무것도 넣을 수 없다.
      - 굳이 넣고 싶으면 Lower-Bounded Wildcards 쓰면 된다.
        - `List<?>`이걸  `List<? super String>` 이렇게 하면 아무 원소나 넣을 수 없으니 불변식을 훼손하지 않고 넣을 수 있다. 

```java
// 코드 26-5 잘못된 예 - 모르는 타입의 원소도 받는 로 타입을 사용했다.
static int numElementsInCommon(Set s1, Set s2) {...}
// 코드 26-6 비한정적 와일드카드 타입을 사용하라. - 타입 안전하며 유연하다.
static int numElementsInCommon(Set<?> s1, Set<?> s2) {...}
```



### 로 타입을 써야하는 예외

- `class` 리터럴에는 로 타입을 써야 한다.
  - 허용 (로타입)
    - `List.class`, `String[].class`, `int.class`
  - 허용하지 않는거 (로 타입 외의 타입)
    - `List<String>.class`, `List<?>.class`
- `instanceof` 연산자와 관련해서
  - `instanceof` 연산자 뒤에는 로 타입과 비한정적 와일드카드 타입만 올 수 있다.
    - 런타임에는 제네릭 타입 정보가 지워지기 때문에 둘 다 똑같이 동작한다.
    - 제네릭 타입이나 한정적 와일드카드 등을 쓰면 컴파일 에러가 난다.
  - 밑에 예제가 제네릭 타입에 `intanceof`를 사용하는 올바른 예다.

```java
// 코드 26-7 로 타입을 써도 좋은 예 - instanceof 연산자
if (o instanceof Set) {     // 로 타입
  	Set<?> s = (Set<?>) o;  // 와일드카드 타입
  	...
}
```



## 아이템 27. 비검사 경고를 제거하라

> In summary, unchecked warnings are important. Don’t ignore them. Every unchecked warning represents the potential for a ClassCastException at run- time. Do your best to eliminate these warnings. If you can’t eliminate an unchecked warning and you can prove that the code that provoked it is typesafe, suppress the warning with a @SuppressWarnings("unchecked") annotation in the narrowest possible scope. Record the rationale for your decision to suppress the warning in a comment.

> 비검사 경고는 중요하니 무시하지 말자. 모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거하라. 경고를 없앨 방법을 찾지 못하겠다면, 그 코드가 타입 안전함을 증명하고 가능한 한 범위를 좁혀 @SuppressWarnings("unchecked") 애너테이션으로 경고를 숨겨라. 그런 다음 경고를 숨기기로 한 근거를 주석으로 남겨라.



- 할 수 있는 한 모든 비검사 경고를 제거하자.
- 경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 애너테이션을 달아 경고를 숨기자.
  - `@SuppressWarnings` 애너테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다. 
  - 하지만 `@SuppressWarnings` 애너테이션은 항상 가능한 한 좁은 범위에 적용하자.
  - `@SuppressWarnings("unchecked")` 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 잠겨야 한다.



## 아이템 28. 배열보다는 리스트를 사용하라

> In summary, arrays and generics have very different type rules. Arrays are covariant and reified; generics are invariant and erased. As a consequence, arrays provide runtime type safety but not compile-time type safety, and vice versa for generics. As a rule, arrays and generics don’t mix well. If you find yourself mixing them and getting compile-time errors or warnings, your first impulse should be to replace the arrays with lists.

> 배열과 제네릭에는 매우 다른 타입 규칙이 적용된다. 배열은 공변이고 실체화되는 반면, 제네릭을 불공변이고 타입 정보가 소거된다. 그 결과 배열은 런타임에는 타입 안전하지만 컴파일타임에는 그렇지 않다. 제네릭은 반대다. 그래서 둘을 섞어 쓰기란 쉽지 않다. 둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용해보자.



### 배열과 제네릭 타입의 차이

- **배열은 공변(covariant)인데, 제네릭은 불공변(invariant)이다.**
  - `Sub`이 `Super`의 하위 타입이라면 `Sub[]`은 `Super[]`의 하위 타입이 된다.
    - 공변, 즉 함께 변한다는 뜻.
  - 서로 다른 타입 `Type1`과 `Type2`가 있을 때, `List<Type1>`은 `List<Type2>`의 하위 타입도 아니고 상위 타입도 아니다
    - `Type1`과 `Type2`가 상속관계에 있다고 해도`List<Type1>`과 `List<Type2>`은 서로 상위 타입도 하위 타입도 아니다.
  - 배열에서는 실수를 런타임에야 알게 되지만 리스트는 컴파일할 때 바로 알 수 있다.

```java
// 코드 28-1 런타임에 실패한다.
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다"; // ArrayStoreException을 던진다.
```

```java
// 코드 28-2 컴파일되지 않는다.
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다.
ol.add("타입이 달라 넣을 수 없다.");
```



- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인하지만 제네릭은 타입 정보가 런타임에는 소거된다.
  - 소거는 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함계 사용할 수 있게 해주는 매커니즘으로, 자바 5가 제네릭으로 순조롭게 전환될 수 있도록 해줬다(아이템 26).
- 이런 주요 차이로 인해 배열과 제네릭은 잘 어우러지지 못한다.



### 제네릭 배열 생성 불허용

- 배열은 제네릭 타입(`new List<E>[]`), 매개변수화 타입(`new List<String>[]`), 타입 매개변수(`new E[]`)로 사용할 수 없다(괄호안이 안되는 예).

```java
List<String>[] stringLists = new List<String>[1];  // (1)
List<Integer> intList = List.of(42);               // (2)
Object[] objects = stringLists;                    // (3)
objects[0] = intList;                              // (4)
String s = stringLists[0].get(0);                  // (5)
```

- (5) `stringLists[0].get()`
  - (3) `List<String>[]`를 `Object[]`에 할당
    - 배열은 공변이니 이게 된다고 가정
  - (4) `List<Integer>` 이거를 `objects[0]` 에
  - `get(0)` 하면 `Integer`가 나온다.
    - 컴파일러는 꺼낸 원소를 자동으로 `String`으로 형변환하는데, 이때 `ClassCastException`이 발생
    - 이런 일을 방지하기 위해서는 제네릭 배열이 생성되지 않도록 (1)에서 컴파일 오류룰 내야 한다.



### 실체화 불가 타입

- `E`(타입 파라미터), `List<E>`(제네릭 타입), `List<String>`(매개변수화 타입) 같은 타입을 **실체화 불가 타입**(non-reifiable type)이라고 한다.

- - 실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다.
  - 배열을 비한정적 와일드카드 타입으로 만들 수는 있지만, 유용하게 쓰일 일은 거의 없다.

- - 실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다.
  - 배열을 비한정적 와일드카드 타입으로 만들 수는 있지만, 유용하게 쓰일 일은 거의 없다.



### `E[]` 대신 `List<E>`를 사용

- 비검사 형변환 경고를 제거하려면 배열 대신 리스트를 쓰면 된다.

```java
// 코드 28-4 Chooser - 제네릭을 시급히 적용해야 한다!
public class Chooser {
  
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
  
}
```

```java
// 코드 28-5 Chooser를 제네릭으로 만들기 위한 첫 시도 - 컴파일되지 않는다.
public class Chooser<T> {
  
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = choices.toArray();
      	// (T[]) choices.toArray();
      	// 이렇게 형변환을 해도 컴파일러는 T가 무슨 타입인지 알 수 없으니
       	// 이 형변환이 런타임에도 안전한지 보장할 수 없다는 경고 메세지를 띄운다. 
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
  
}
```

```java
// 코드 28-6 리스트 기반 Chooser - 타입 안전성 확보!
public class Chooser<T> {
  
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
  
}
```



## 아이템 29. 이왕이면 제네릭 타입으로 만들라

> In summary, generic types are safer and easier to use than types that require casts in client code. When you design new types, make sure that they can be used without such casts. This will often mean making the types generic. If you have any existing types that should be generic but aren’t, generify them. This will make life easier for new users of these types without breaking existing clients (Item 26).

> 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다. 그러니 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라. 그렇게 하려면 제네릭 타입으로 만들어야 할 경우가 많다. 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하자. 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해주는 길이다(아이템 26).



- 코드 29-1 `Stack` 클래스는 원래 제네릭 타입이어야 마땅하다.

  - 제네릭으로 만들어보자.
  - 이 클래스를 제네릭으로 바꾼다고 해도 현재 버전을 사용하는 클라이언트에는 아무런 해가 없다.

- 일반 클래스를 제네릭 클래스로 만드는 첫 단계는 클래스 선언에 **타입 파라미터**를 추가하는 일이다.

  - ```java
    elements = new E[DEFAULT_INITIAL_CAPACITY];
    ```

    - `E`와 같은 타입 파라미터는 실체화 불가 타입으로 배열을 만들 수 없다(아이템 28).
      - 해결책 두 가지

- 첫 번째 방법은 제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법

  - `Object` 배열을 생성한 다음 제네릭 배열로 형변환
    - 배열 `elements`는 `private` 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없다.
    - `push` 메서드를 통해 배열에 저장되는 원소의 타입은 항상 `E`다.
    - 이 비검사 형변환은 확실히 안전하다.
    - 비검사 형변환이 안전함을 직접 증명했다면 범위를 최소로 좁혀 `@SuppressWarnings` 애너테이션으로 해당 경고를 숨긴다(아이템 27).

```java
// 코드 29-2 제네릭 스택으로 가는 첫 단계 - 컴파일되지 않는다.
// E[]를 이용한 제네릭 스택
public class Stack<E> {
  
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    // 코드 29-3 배열을 사용한 코드를 제네릭으로 만드는 방법 1
    // 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
    // 따라서 타입 안전성을 보장하지만,
    // 이 배열의 런타임 타입은 E[]가 아닌 Object[]다!
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    ...
  
}
```



- 두 번째 방법은 `elements` 필드의 타입을 `E[]`에서 `Object[]`로 바꾸는 것이다.
  - 배열이 반환한 원소를 E로 형변환하면 오류 대신 경고가 뜬다.
    - pop 메서드 전체에서 경고를 숨기지 말고, 아이템 27의 조언을 따라 비검사 형변환을 수행하는 할당문에서만 숨기는 부분 체크

```java
// Object[]를 이용한 제네릭 Stack
public class Stack<E> {
  
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    // 코드 29-4 배열을 사용한 코드를 제네릭으로 만드는 방법 2
    // 비검사 경고를 적절히 숨긴다.
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();

        // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
        // 경고를 숨기는 범위를 이 정도로 줄일 수가 있구나.
        @SuppressWarnings("unchecked") E result =
                (E) elements[--size];

        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

		...
  
}
```



- 제네릭 배열 생성을 제거하는 두 방법 모두 나름의 지지를 얻고 있다.
  - 첫 번째 방법은 가독성이 더 좋다.
    - 배열의 타입을 `E[]`로 선언하여 오직 `E` 타입 인스턴스만 받음을 확실히 어필한다.
  - 첫 번째 방식에서는 형변환을 배열 생성 시 다 한 번만 해주면 된다.
    - 하지만 두 번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야 한다. 
    - 따라서 현업에서는 첫 번째 방식을 더 선호하며 자주 사용한다.
  - (`E`가 `Object`가 아닌 한) 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염(`heap pollution`; 아이템 32)을 일으킨다.
    - 힙 오염이 맘에 걸리는 프로그래머는 두 번째 방식을 고수하기도 한다(이번 아이템의 예에서는 힙 오염이 해가 되지 않았다).



- 지금까지 설명한 `Stack` 예는 "배열보다는 리스트를 우선하라"는 아이템 28과 모순돼 보인다. 
  - 사실 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아니다.
  - 자바가 리스트를 기본 타입으로 제공하지 않으므로 `ArrayList` 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 한다.
  - `HashMap` 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.



## 아이템 30. 이왕이면 제네릭 메서드로 만들라

> In summary, generic methods, like generic types, are safer and easier to use than methods requiring their clients to put explicit casts on input parameters and return values. Like types, you should make sure that your methods can be used without casts, which often means making them generic. And like types, you should generify existing methods whose use requires casts. This makes life easier for new users without breaking existing clients (Item 26).

> 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다. 타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우  그렇게 하려면 제네릭 메서드가 되어야 한다. 역시 타입과 마찬가지로, 형변환을 해줘야 하는 기존 메서드는 제네릭하게 만들자. 기존 클라이언트는 그대로 둔 채 새로운 사용자의 삶을 훨씬 편하게 만들어줄 것이다(아이템 26). 



- 제네릭 메서드 작성법은 제네릭 타입 작성법과 비슷하다.

  - (타입 매개변수들을 선언하는) **타입 매개변수 목록**은 메서드의 제한자와 반환 타입 사이에 온다.

    - ```java
      public static <E> Set<E> union(Set<E> s1, Set<E> s2) {...}
      ```

      - 다음 코드에서 타입 매개변수 목록은 `<E>`이고 반환 타입은 `Set<E>`이다.

    - 타입 매개변수의 명명 규칙은 제네릭 메서드나 제네릭 타입이나 똑같다(아이템 29, 68).



### 제네릭 싱글턴 팩터리

- 제네릭 싱글턴 팩터리
  - 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리 패턴
- 항등함수
  - `T`인자를 받으면 `T` 객체를 반환하는 함수 인터페이스
  - 항등함수 객체는 상태가 없으니 요청할 때마다 새로 생성하는 것은 낭비다.
  - 자바의 제네릭이 실체화된다면 항등함수를 타입별로 하나씩 만들어야 했겠지만, 소거 방식을 사용한 덕에 제네릭 싱글턴 하나면 충분하다.

```java
// 제네릭 싱글턴 팩터리 패턴
public class GenericSingletonFactory {
  
    // 코드 30-4 제네릭 싱글턴 팩터리 패턴
  	// 여기서는 타입을 Object
    private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        // 반환할 때 형변환
        // 여기서 리턴타입이 UnaryOperator<Object>라면 받는 쪽에서 형변환 해줘야한다.
      	return (UnaryOperator<T>) IDENTITY_FN;
    }

    // 코드 30-5 제네릭 싱글턴을 사용하는 예
    public static void main(String[] args) {
        String[] strings = { "삼베", "대마", "나일론" };
      	// 리턴타입이 UnaryOperator<T>이기 때문에 원하는 타입으로 받을 수 있다. 
        UnaryOperator<String> sameString = identityFunction();
        for (String s : strings)
            System.out.println(sameString.apply(s));

        Number[] numbers = { 1, 2.0, 3L };
        UnaryOperator<Number> sameNumber = identityFunction();
        for (Number n : numbers)
            System.out.println(sameNumber.apply(n));
    }
  
}
```



### 재귀적 타입 한정

- 재귀적 타입 한정(recursive type bound)

  - 상대적으로 드물긴 하지만, 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.

  - 재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 `Comparable` 인터페이스(아이템 14)와 함께 쓰인다.

    - `Comparable`을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그 원소들을 정렬 혹은 검색하거나, 최솟값이나 최대값을 구하는 식으로 사용된다. 

    - 이 기능을 수행하려면 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 한다.

      - ```java
        public static <E extends Comparable<E>> E max(Collection<E> c);
        ```

        - 재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현했다.

```java
// 재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현
public class RecursiveTypeBound {
  
    // 코드 30-7 컬렉션에서 최댓값을 반환한다. - 재귀적 타입 한정 사용
    public static <E extends Comparable<E>> E max(Collection<E> c) {
        if (c.isEmpty())
            throw new IllegalArgumentException("컬렉션이 비어 있습니다.");

        E result = null;
        for (E e : c)
            if (result == null || e.compareTo(result) > 0)
                result = Objects.requireNonNull(e);

        return result;
    }

    public static void main(String[] args) {
        List<String> argList = Arrays.asList(args);
        System.out.println(max(argList));
    }
  
}
```



## 아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

> In summary, using wildcard types in your APIs, while tricky, makes the APIs far more flexible. If you write a library that will be widely used, the proper use of wildcard types should be considered mandatory. Remember the basic rule: producer-extends, consumer-super (PECS). Also remember that all comparables and comparators are consumers.

> 조금 복잡하더라도 와일드 카드 타입을 적용하면 API가 훨씬 유연해진다. 그러니 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 절절히 사용해줘야 한다. PECS 공식을 기억하자. 즉, 생산자(producer)는 extends를 소비자(consumer)는 super를 사용한다. Comparable과 Comparator는 모두 소비자라는 사실도 잊지 말자.



### 한정적 와일드카드 타입

- 아이템 28에서 이야기했듯 매개변수화 타입은 불공변(invariant)이다.
  - 서로 다른 타입 `Type1`과 `Type2`가 있을 때 `List<Type1>`은 `List<Type2>`의 하위 타입도 상위 타입도 아니다.
  - `List<Object>`에는 어떤 객체든 넣을 수 있지만 `List<String>`에는 문자열만 넣을 수 있다.
    - `List<String>`은 `List<Object>`가 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다.
      - 리스코프 치환 원칙에 어긋난다(아이템 10).



```java
// 코드 31-1 와일드카드 타입을 사용하지 않은 pushAll 메서드 - 결함이 있다!
public void pushAll(Iterable<E> src) {
    for (E e : src)
      push(e);
}
```

- `Stack<Number>`로 선언한 후 `pushAll(intVal)`을 호출
  - `intVal`은 `Integer` 타입
  - `Integer`는 `Number`의 하위 타입이니 잘 동작해야 할 것 같다.
  - 실제로는 오류 메시지가 뜬다.
    - incompatible types
    - 매개변수화 타입이 불공변이기 때문



```java
// 코드 31-2 E 생산자(producer) 매개변수에 와일드카드 타입 적용
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

- `pushAll`의 입력 매개변수 타입은 '`E`의 `Iterable`'이 아니라 '`E`의 하위 타입의 `Iterable`'이어야 한다.
  - 와일드카드 타입 `Iterable<? extends E>`가 정확히 이런 뜻이다. 



```java
// 코드 31-3 와일드카드 타입을 사용하지 않은 popAll 메서드 - 결함이 있다!
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

- 코드 31-3을 컴파일하면 "`Collection<Object>`는 `Collection<Number>`의 하위 타입이 아니다"라는 오류가 발생한다.



```java
// 코드 31-4 E 소비자(consumer) 매개변수에 와일드카드 타입 적용
public void popAll(Collection<? super E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
```

- 이번에는 `popAll`의 입력 매개변수의 타입이 '`E`의 `Collection`'이 아니라 '`E`의 상위 타입의 `Collection`'이어야 한다.
  - 와일드카드 타입을 사용한 `Collection<? super E>`가 정확히 이런 의미다.



- 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.
  - 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다.
    - 타입을 정확히 지정해야 하는 상황으로, 이때는 와일드카드 타입을 쓰지 말아야 한다.



### 펙스(PECS)

- producer-extends, consumer-super
  - **매개변수화 타입 T가 생산자라면 `<? extends T>`를 사용하고, 소비자라면 `<? super T>`를 사용하자.**

```java
// 변경 전
public Chooser(Collection<T> choices) {...}

// 코드 31-5 T 생산자 매개변수에 와일드카드 타입 적용
public Chooser(Collection<? extends T> choices) {...}


// 변경 전
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {...}

// 코드 30-2의 제네릭 union 메서드에 와일드카드 타입을 적용해 유연성을 높였다.
public static <E> Set<E> union(Set<? extends E> s1,
                               Set<? extends E> s2) {...}


// 변경 전
public static <E extends Comparable<E>> E max(List<E> list) {...}

// 변경 후

// 입력 매개변수에서는 E 인스턴스를 생산하므로 원래의 List<E>를 List<? extends E>로 수정

// Comparable(혹은 Comparator)을 직접 구현하지 않고, 
// 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드카드 
// <E extends Comparable<? super E>> 이게 필요하다

public static <E extends Comparable<? super E>> E max(List<? extends E> list) {...}
```



### 타입 파라미터와 와일드카드 공통되는 부분

- 타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다.
  - 예를 들어 주어진 리스트에서 명시한 두 인덱스의 아이템들을 교환(swap)하는 정적 메서드를 두 방식 모두로 정의해보자.
    - 코드 31-7에서 첫 번째는 비한정적 타입 매개변수(아이템 30)를 사용했고 두 번째는 비한정적 와일드카드를 사용했다.

```java
// 코드 31-7 swap 메서드의 두 가지 선언
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

- 어떤 선언이 나을까? 더 나은 이유는 무엇일까?
  - public API라면 간단한 두 번째가 낫다.
- 기본 규칙은 이렇다.
  - 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하라.
    - 이때 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고
    - 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.

```java
public static void swap(List<?> list, int i, int j) {
  	list.set(i, list.set(j, list.get(i)));
}
```

- 이 swap 선언은 컴파일하면 오류 메시지가 나온다.
  - 방금 꺼낸 원소를 리스트에 다시 넣을 수 없다니 왜 그럴까?
    - 리스트의 타입이 `List<?>`인데, `List<?>`에는 null 외에는 어떤 값도 넣을 수 없다.
      - 와일드카드 타입의 실제 타입을 알려주는 메서드를 **private 도우미 메서드**로 따로 작성하여 활용해보자.

```java
// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
public class Swap {
    public static void swap(List<?> list, int i, int j) {
        swapHelper(list, i, j);
    }

    // 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
    private static <E> void swapHelper(List<E> list, int i, int j) {
        list.set(i, list.set(j, list.get(i)));
    }

    public static void main(String[] args) {
        // 첫 번째와 마지막 인수를 스왑한 후 결과 리스트를 출력한다.
        List<String> argList = Arrays.asList(args);
        swap(argList, 0, argList.size() - 1);
        System.out.println(argList);
    }
}
```




## 아이템 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라

> In summary, varargs and generics do not interact well because the varargs facility is a leaky abstraction built atop arrays, and arrays have different type rules from generics. Though generic varargs parameters are not typesafe, they are legal. If you choose to write a method with a generic (or parameterized) varargs parame- ter, first ensure that the method is typesafe, and then annotate it with @SafeVarargs so it is not unpleasant to use.

> 가변인수와 제네릭은 궁합이 좋지 않다. 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입 규칙이 서로 다르기 때문이다. 제네릭 varargs 매개변수는 타입 안전하지는 않지만, 허용된다. 메서드에 제네릭 (혹은 매개변수화된) varargs 매개변수를 사용하고자 한다면, 먼저 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs 애너테이션을 달아 사용하는 데 불편함이 없게끔 하자.



- 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다.
  - 그런데 내부로 감춰야 했을 이 배열을 그만 클라이언트에 노출하는 문제가 생겼다.
  - 그 결과 `varargs` 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다.
    - 호출자 쪽에 경고 뜬다.
      -  `Unchecked generics array creation for varargs parameter `
- 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.
  - 타입 안정성이 깨지니 제네릭 `varargs` 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

```java
// 코드 32-1 제네릭과 varargs를 혼용하면 타입 안정성이 깨진다!
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intLists; // 힙 오염 발생
    String s = stringLists[0].get(0); // ClassCastException
}
```



- 제네릭 배열을 프로그래머가 직접 생성하는 건 허용하지 않으면서 제네릭 `varargs` 매개변수를 받는 메서드를 선언할 수 있게 한 이유는 실무에서 매우 유용하기 때문임.
  - `Arrays.asList(T… a)`, `Collections.addAll(Collection<? Super T> c, T… elements)` 등



- 자바 7 전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해서 해줄 수 있는 일이 없었다.
  - 사용자는 이 경고를 그냥 두거나 (더 흔하게는) 호출하는 곳마다 `@SuppressWarnings("unchecked")` 애너테이션을 달아 경고를 숨겨야 했다(아이템 27).
- 자바 7에서는 `@SafeVarargs` 애너테이션이 추가되어 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다.
  - `@SafeVarargs` 애너테이션은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치다.
- `varargs` 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면(`varargs`의 목적대로만 쓰인다면) 그 메서드는 안전하다.
  - 메서드 안에서 배열에 아무것도 저장하지 않고, 배열의 참조가 밖으로 노출되지 않는다면 타입 안전하다.



- `varargs` 매개변수 배열에 아무것도 저장하지 않고도 타입 안전성을 깰수도 있으니 주의해야 한다.
  - 이 메서드가 반환하는 배열의 타입은 이 메서드에 인수를 넘기는 컴파일타임에 결정되는데, 그 시점에는 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있다.
  - 따라서 자신의 varargs 매개변수 배열을 그대로 반환하면 힙 오염을 이 메서드를 호출한 쪽의 콜스택으로까지 전이하는 결과를 낳을 수 있다.
  - 밑에 코드를 보면 pickTwo 메서드를 본 컴파일러는 `Object[]`을 만들어 준다.
    - 안에서 호출하는 toArray 메서드가 가변인수라서 배열을 만들어야하는데 자신이(picktwo) 받는 매개변수 타입이 타입 파라미터이기 때문에 어떤 타입도 다 담을 수 있는 `Object` 타입 배열을 만든다.
    - pickTwo는 toArray에서 반환 받은 `Object[]`을 그대로 리턴하고 있다.
    - 근데 pickTwo를 호출 할 때는 `String` 타입을 넘기네?
      - 컴파일러는 이 부분을 보고 `String[]`을 만들어 놓고 있는데 반환이 `Object[]`이 오네?
        - `Object`는 `String`의 상위 타입이기 때문에 `String`에서 `Object`로는 형변환이 안된다.
        - `ClassCastException`

```java
// 미묘한 힙 오염 발생
public class PickTwo {
    // 코드 32-2 자신의 제네릭 매개변수 배열의 참조를 노출한다. - 안전하지 않다!
    static <T> T[] toArray(T... args) {
        return args;
    }

    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); // 도달할 수 없다.
    }

    public static void main(String[] args) {
        String[] attributes = pickTwo("좋은", "빠른", "저렴한");
        System.out.println(Arrays.toString(attributes));
    }
}
```



- 위 예는 제네릭 `varargs` 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다는 점을 다시 한번 상기시킨다.
  - 단, 예외가 두 가지 있다.
    - 첫 번째, `@SafeVarargs`로 제대로 애노테이트된 또 다른 `varargs` 메서드에 넘기는 것은 안전하다.
    - 두 번째, 그저 이 배열 내용의 일부 함수를 호출만 하는(`varargs`를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
      	// 모든 원소를 하나의 리스트로 옮겨 담아 반환
      	result.addAll(list);
    return result;
}
```



- 제네릭이나 매개변수화 타입의 `varargs` 매개변수를 받는 모든 메서드에 `@SafeVarargs`를 달라.
  - 이 말은 안전하지 않은 `varargs` 메서드는 절대 작성해서는 안 된다는 뜻이다.
- 통제할 수 있는 메서드 중 제네릭 `varargs` 매개변수를 사용하며 힙 오염 경고가 뜨는 메서드가 있다면, 그 메서드가 진짜 안전한지 점검하라.
  - 다음 두 조건을 모두 만족하는 제네릭 `varargs` 메서드는 안전하다. 둘 중 하나라도 어겼다면 수정하자.
    - `varargs` 매개변수 배열에 아무것도 저장하지 않는다.
    - 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.



- `@SafeVarargs` 애너테이션은 재정의할 수 없는 메서드에만 달아야 한다.
  - 재정의한 메서드도 안전할지는 보장할 수 없기 때문이다.
  - 자바 8에서 이 애너테이션은 오직 정적 메서드와 `final` 인스턴스 메서드에만 붙일 수 있다.
  - 자바 9부터는 `private` 인스턴스 메서드에도 허용된다.



- `@SafeVarargs` 애너테이션이 유일한 정답은 아니다.
  - 아이템 28의 조언을 따라(실체는 배열인) `varargs` 매개변수를 `List` 매개변수로 바꿀 수도 있다.
    - `List.of`에도 `@SafeVarargs` 애너테이션이 달려 있다.
    - 장점
      - 컴파일러가 이 메서드의 타입 안전성을 검증할 수 있다는 데 있다.
      - `@SafeVararsg` 애너테이션을 우리가 직접 달지 않아도 된다.
      - 실수로 안전하다고 판단할 걱정도 없다.
    - 단점
      - 클라이언트 코드가 살짝 지저분해지고 속도가 조금 느려질 수 있다는 정도다.

```java
// 코드 32-4 제네릭 varargs 매개변수를 List로 대체한 예 - 타입 안전하다.
public class FlattenWithList {
    static <T> List<T> flatten(List<List<? extends T>> lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists)
            result.addAll(list);
        return result;
    }

    public static void main(String[] args) {
        List<Integer> flatList = flatten(List.of(
                List.of(1, 2), List.of(3, 4, 5), List.of(6,7)));
        System.out.println(flatList);
    }
}
```



## 아이템 33. 타입 안전 이종 컨테이너를 고려하라

> In summary, the normal use of generics, exemplified by the collections APIs, restricts you to a fixed number of type parameters per container. You can get around this restriction by placing the type parameter on the key rather than the container. You can use Class objects as keys for such typesafe heterogeneous containers. A Class object used in this fashion is called a type token. You can also use a custom key type. For example, you could have a DatabaseRow type repre- senting a database row (the container), and a generic type Column<T> as its key.

> 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다. 하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다. 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다. 또한, 직접 구현한 키 타입도 쓸 수 있다. 예컨대 데이터베이스의 행(컨테이너)을 표현한 DatabaseRow 타입에는 제네릭 타입인 Column<T>를 키로 사용할 수 있다.



### 타입 안전 이종 컨테이너

- 데이터베이스의 행(row)은 임의 개수의 열(column)을 가질 수 있는데, 모두 열을 타입 안전하게 이용할 수 있다면 멋질 것이다.
  - 다행히 쉬운 해법이 있다.
  - 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다.
    - 이렇게 하면 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해줄 것이다.
    - 이러한 설계 방식을 **타입 안전 이종 컨테이너 패턴**(type safe heterogeneous pattern)이라 한다.



- 타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorites 클래스를 생각해보자.
  - **각 타입의 Class 객체**를 매개변수화한 키 역할로 사용하면 되는데, 이 방식이 동작하는 이유는 **class의 클래스가 제네릭**이기 때문이다.
    - class 리터럴의 타입은 Class가 아닌 Class<T>다.
      - String.class의 타입은 Class<String>, Integer.class의 타입은 Class<Integer>
- 타입 토큰(type token)
  - String.class, Integer.class 등 이런 형태가 타입 토큰임.
  - 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴



```java
// 타입 안전 이종 컨테이너 패턴
public class Favorites {
  
    // 코드 33-3 타입 안전 이종 컨테이너 패턴 - 구현
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }

//    // 코드 33-4 동적 형변환으로 런타임 타입 안전성 확보
//    public <T> void putFavorite(Class<T> type, T instance) {
//        favorites.put(Objects.requireNonNull(type), type.cast(instance));
//    }

    // 코드 33-2 타입 안전 이종 컨테이너 패턴 - 클라이언트
    public static void main(String[] args) {
        Favorites f = new Favorites();
        
        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebabe);
        f.putFavorite(Class.class, Favorites.class);
       
        String favoriteString = f.getFavorite(String.class);
        int favoriteInteger = f.getFavorite(Integer.class);
//        String a =  f.getFavorite(Integer.class);

        Class<?> favoriteClass = f.getFavorite(Class.class);
        
        System.out.printf("%s %x %s%n", favoriteString,
                favoriteInteger, favoriteClass.getName());
    }
  
}
```

- Favorites 인스턴스는 타입 안전하다.
  - String을 요청했는데 Integer를 반환하는 일은 절대 없다.
    - `(Class<T> type, T instance)`
      - 둘 다 T, 타입 파라미터가 그걸 보장해 주고 있다.
  - 모든 키의 타입이 제각각이라, 일반적인 맵과 달리 여러 가지 타입의 원소를 담을 수 있다.
  - Favorites는 타입 안전 이종 컨테이너라 할 만하다.

- Favorites가 사용하는 private 맵 변수인 favorites의 타입은 Map<Class<?>, Object>이다.
  - 비한정적 와일드카드 타입이라 이 맵 안에 아무것도 넣을 수 없다고 생각할 수 있지다.
    - 하지만 사실은 그 반대다.
    - 와일드카드 타입이 중첩(nested)되었다는 점에 주목해야 한다.
      - 맵이 아니라 키가 와일드카드 타입인 것이다.
        - 키로 아무 타입이나 다 넣을 수 있게 된거다.



- Favorites 클래스 제약 두 가지

  - 첫 번째, 악의적인 클라이언트가 Class 객체를 (제네릭이 아닌) 로 타입(아이템 26)으로 넘기면 Favorites 인스턴스의 타입 안전성이 쉽게 깨진다.

    - ```java
      f.putFavorite((Class)Integer.class, "Integer의 인스턴스가 아닙니다.");
      ```

      - `Class` 타입으로 캐스팅해서 넣으니깐 들어가네.

    ```java
    // 코드 33-4 동적 형변환으로 런타임 타입 안전성 확보 (202쪽)
    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
    ```

    - 불변식을 보장하려면 putFavorite 메서드에서 인수로 주어진 instance의  타입이 type으로 명시한 타입과 같은지 확인하면 된다.

  - 두 번째, 실체화 불가 타입(아이템 28)에는 사용할 수 없다. 

    - String이나 String[]은 저장할 수 있어도 List<String>은 컴파일 되지 않는다.
      - List<String>용 Class 객체를 얻을 수 없기 때문이다.
    - 슈퍼 타입 토큰
    - 허용하는 타입을 제한하고 싶다면 한정적 타입 토큰을 활용하면 가능하다.
      - Class<?> 타입의 객체가 있고, 이를 한정적 타입 토큰을 받는 메서드에 넘기려면 어떻게 해야 할까?
        - Class 클래스가 이런 형변환을 안전하게 (그리고 동적으로) 수행해주는 asSubclass 인스턴스 메서드를 제공한다.
        - 코드 33-5는 컴파일 시점에는 타입을 알 수 없는 애너테이션을 asSubclass 메서드를 사용해 런타임에 읽어낸다.

```java
// 코드 33-5 asSubclass를 사용해 한정적 타입 토큰을 안전하게 형변환한다.
public class PrintAnnotation {
    static Annotation getAnnotation(AnnotatedElement element,
                                    String annotationTypeName) {
      
        Class<?> annotationType = null; // 비한정적 타입 토큰
        try {
            annotationType = Class.forName(annotationTypeName);
        } catch (Exception ex) {
            throw new IllegalArgumentException(ex);
        }
        return element.getAnnotation(
                annotationType.asSubclass(Annotation.class));
      
    }
  
		...
```