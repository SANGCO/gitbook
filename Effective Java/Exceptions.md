

## 아이템 69. 예외는 진짜 예외 상황에만 사용하라

> In summary, exceptions are designed for exceptional conditions. Don’t use them for ordinary control flow, and don’t write APIs that force others to do so.

> 예외는 예외 상황에서 쓸 의도로 설계되었다. 정상적인 제어 흐름에서 사용해서는 안 되며, 이를 프로그래머에게 강요하는 API를 만들어서도 안 된다.




```java
// 코드 69-1 예외를 완전히 잘못 사용한 예 - 따라 하지 말 것!
try {
  int i = 0;
  while(true)
    range[i++].climb();
} catch(ArrayIndexOutOfBoundsException e) {}

// 위 코드를 표준적인 관용구대로 다시 작성
for (Mountain m : range)
  m.climb();
```

- 무한루프를 돌다가 배열의 끝에 도달해 `ArrayIndexOutOfBoundsException`이 발생하면 끝나는 코드 69-1 예제
  - 잘못된 추론을 근거로 성능을 높여보려했다.
    - `JVM`은 배열에 접근할 때마다 경계를 넘지 않는지 검사한다.
      -  일반적인 반복문도 경계를 넘지 않는지 검사하고 배열 경계에 도달하면 종료된다.
        - 코드 69-1 예제는 박복문이 하는 이 검사를 생략하려고 했다.
  - 세 가지 면에서 잘못된 추론이다.
    - 예외는 예외 상황에 쓸 용도로 설계되었으므로 `JVM` 구현자 입장에서는 명확한 검사만큼 빠르게 만들어야 할 동기가 약하다(**최적화**에 별로 신경 쓰지 않았을 가능성이 크다).
    - 코드를 `try-catch` 블록 안에 넣으면 `JVM`이 적용할 수 있는 최적화가 제한된다.
    - 배열을 순회하는 표준 관용구는 앞서 걱정한 중복 검사를 수행하지 않는다. `JVM`이 알아서 최적화해 없애준다.
  - 실상은 예외를 사용한 쪽이 표준 관용구보다 훨씬 느리다.



- 예외는 (그 이름이 말해주듯) 오직 예외 상황에서만 써야 한다. 절대로 일상적인 제어 흐름용으로 쓰여선 안 된다.
  - 표준적이고 쉽게 이해되는 관용구를 사용하고, 성능 개선을 목적으로 과하게 머리를 쓴 기법은 자제하라.
    - 실제로 성능이 좋아지더라도 자바 플랫폼이 꾸준히 개선되고 있으니 최적화로 얻은 상대적인 성능 우위가 오래가지 않을 수 있다.
    - 반면 과하게 영리한 기법에 숨겨진 미묘한 버그의 폐해와 어려워진 유지보수 문제는 계속 이어질 것이다.

- 잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야 한다.
  - 특정 상태에서만 호출할 수 있는 '**상태 의존적**' 메서드를 제공하는 클래스는 '**상태 검사**' 메서드도 함계 제공해야 한다. 
    - `Iterator 인터페이스`의 `next`와 `hasNext`가 각각 상태 의존적 메서드와 상태 검사 메서드에 해당한다.
      - `for-each`도 내부적으로 `hasNext`를 사용한다.)

```java
for (Iterator<Foo> i = collection.iterator(); i.hasNext(); ) {
  Foo foo = i.next();
  ...
}

// Iterator가 hasNext를 제공하지 않았다면 그 일을 클라이언트가 대신해야만 했다.
// 컬렉션을 이런 식으로 순회하지 말 것!
try {
  Iterator<Foo> i = collection.iterator();
  while(true) {
    Foo foo = i.next();
    ...
  }
} catch(NoSuchElementException e) {
}
```



- 상태 검사 메서드 대신 사용할 수 있는 선택지도 있다.
  - 올바르지 않은 상태일 때 빈 옵셔널(아이템 55) 혹은 **null 같은 특수한 특정 값**을 반환하는 방법이다.
- 상태 검사 메서드, 옵셔널, 특정 값 중 하나를 선택하는 지침을 몇 개 소개하겠다.
  - 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인으로 **상태가 변할 수 있다면** **옵셔널이나 특정 값**을 사용한다.
    - 상태 검사 메서드와 상태 의존적 메서드 호출 사이에 객체의 상태가 변할 수 있기 때문이다.
  - **성능이 중요한 상황**에서 상태 검사 메서드가 상태 의존적 메서드의 작업 일부를 중복 수행한다면 **옵셔널이나 특정 값**을 선택한다.
  - **다른 모든 경우엔 상태 검사 메서드** 방식이 조금 더 낫다고 할 수 있다.
    - 가독성이 살짝 더 좋고, 잘못 사용했을 때 발견하기가 쉽다.
    - 상태 검사 메서드 호출을 깜빡 잊었다면 상태 의존적 메서드가 예외를 던져 버그를 확실히 드러낼 것이다.



## 아이템 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류는 런타임 예외를 사용하라

> To summarize, throw checked exceptions for recoverable conditions and unchecked exceptions for programming errors. When in doubt, throw unchecked exceptions. Don’t define any throwables that are neither checked exceptions nor runtime exceptions. Provide methods on your checked exceptions to aid in recovery.


> 복구할 수 있는 상황이면 검사 예외를, 프로그래밍 오류라면 비검사 예외를 던지자. 확실하지 않다면 비검사 예외를 던지자. 검사 예외도 아니고 런타임 예외도 아닌 throwable은 정의하지도 말자. 검사 예외라면 복구에 필요한 정보를 알려주는 메서드도 제공하자.



### 자바 Exception

- 자바에서 문제 상황을 알리는 타입(`throwable`) 세가지
  - **Checked Exception**
  - **Runtime Exception**
  - **Error**
- 비검사 `throwable`은 두 가지
  - 에러
  - 런타임 예외
    - 런타임 예외는 비검사 예외이다.
- 검사 예외와 비검사 예외
  - 검사 예외는 사용자의 실수와 같은 외적인 요인에 의해 발생하는 예외
  - 비검사 예외는 프로그래머의 실수로 발생하는 예외
- 사용자 정의 `Exception`
  - `Exception`을 `extends`하면 Checked Exception 
  - `RuntimeException`을 `extends`하면 Unchecked Exception
- `Object`
  - `Throwable`
    - `Error` (Unchecked Exception)
    - `Exception` (Checked Exception)
      - `RuntimeException` (Unchecked Exception)



- This constructor of `Scanner` declares `FileNotFoundException` exception, because we assume that the specified file may not exist.
  - Most importantly, there is a single line in the method that **may throw an exception**, so we put the `throws` keyword in the method declaration.

```java
public static String readLineFromFile() throws FileNotFoundException {
    Scanner scanner = new Scanner(new File("file.txt")); // java.io.FileNotFoundException
    return scanner.nextLine();
}
```



- Here is a method that throws `NumberFormatException` when input string has an invalid format (e.g. `"abc"`).
  - This code always successfully compiles without the `throws` keyword in the declaration.

```java
public static Long convertStringToLong(String str) {
    return Long.parseLong(str); // It may throw NumberFormatException
}
```



- 예외 선택하기
  - `Error`
    - 코드로 해결이 안되는 로직 외의 상황
  - `Unchecked Exception`
    - 실시간으로 처리할 수 없는 상황
    - 버그
  - `Checked Exception`
    - 실시간으로 복구할 수 있는 상황
    - 예측되는 예외 상황



### 검사 예외, 비검사 예외

- 호출하는 쪽에서 복구하리라 여겨지는 상황이면 검사 예외를 사용
  - 검사 예외와 비검사 예외를 구분하는 **기본 규칙**
  - 달리 말하면, API 설계자는 API 사용자에게 검사 예외를 던져주어 **그 상황에서 회복해내라고 요구**한 것이다.
  - 검사 예외를 던지면 호출자가 그 예외를 `catch`로 잡아 처리하거나 더 바깥으로 전파하도록 강제하게 된다.



- 비검사 `throwable` **런타임 예외**와 **에러**
  - 둘 다 동작 측면에서는 다르지 않다.
  - 프로그램에서 잡을 필요가 없거나 혹은 통상적으로는 잡지 말아야 한다.
    - 프로그램에서 비검사 예외나 에러를 던졌다는 것은 **복구가 불가능하거나 더 실행해봐야 득보다는 실이 많다는 뜻**이다.
  - 이런 throwable을 잡지 않은 스레드는 적절한 오류 메시지를 내뱉으며 중단된다.
    - 이런 상황이 생기면 프로그램이 중단되는게 맞다고 생각하면 비검사로 가야겠군.



- **프로그래밍 오류를 나타날 때는 런타임 예외를 사용**하자.

  - 런타임 예외의 대부분은 전제조건을 만족하지 못했을 때 발생한다.
    - **전제조건 위배**란 단순히 클라이언트가 해당 API의 명세에 기록된 제약을 지키지 못했다는 뜻이다.
    - 예컨대 배열의 인덱스는 0에서 '배열 크기 -1' 사이여야 한다. `ArrayIndexOutOfBoundsException`이 발생했다는 건 이 전제조건이 지켜지지 않았다는 뜻이다. 

  

- **복구할 수 있는 상황**인지 **프로그래밍 오류**인지가 항상 명확히 구분되지는 않는다.

  - API 설계자의 판단에 따라 복구 가능하다고 믿는다면 검사 예외를, 그렇지 않다면 런타임 예외를 사용하자.
  - **확신하기 어렵다면 아마도 비검사 예외를 선택**하는 편이 나을 것이다(이유는 아이템 71).



### 그 외 체크할 부분

- 에러는 보통 `JVM`이 자원 부족, 불변식 깨짐 등 더 이상 수행을 계속할 수 없는 상황을 나타낼 때 사용
  - `Error` 클래스를 상속해 하위 클래스를 만드는 일은 자제해라.
    - 자바 언어 명세가 요구하는 것은 아니지만 업계에 널리 퍼진 규약
  - 다시 말해 여러분이 구현하는 비검사 `throwable`은 모두 `RuntimeException`의 하위 클래스여야 한다(직접적이든 간접적이든).
  - `Error`는 상속하지 말아야 할 뿐 아니라, throw 문으로 직접 던지는 일도 없어야 한다(`AssertionError`는 예외다).



- `Exception`, `RuntimeException`, `Error`를 상속하지 않는 `throwable`을 만들 수도 있다.
  - 하지만 이로울 게 없으니 절대로 사용하지 말자!
  - `throwable`은 정상적인 검사 예외보다 나을 게 하나도 없으면서 API 사용자를 헷갈리게 할 뿐이다.



- API 설계자들도 예외 역시 어떤 메서드라도 정의할 수 있는 **완벽한 객체**라는 사실을 잊곤 한다.
  - 예외의 메서드는 주로 그 예외를 일으킨 상황에 관한 정보를 코드 형태로 전달하는 데 쓰인다.
  - 이런 메서드가 없다면 프로그래머들은 오류 메시지를 파싱해 정보를 빼내야 하는데, 대단히 나쁜 습관이다(아이템 12).



- 검사 예외는 일반적으로 복구할 수 있는 조건일 때 발생
  - 따라서 호출자가 예외 상황에서 벗어나는 데 필요한 정보를 알려주는 메서드를 함께 제공하는 것이 중요
  - 예컨대 쇼핑몰에서 물건을 구입하려는 데 카드 잔고가 부족하여 검사 예외가 발생했다고 해보자.
    - 그렇다면 이 예외는 잔고가 얼마나 부족한지를 알려주는 접근자 메서드르 제공해야 한다.
      - 이 주제는 아이템 75에서 더 이야기할 것이다.



## 아이템 71. 필요 없는 검사 예외 사용은 피하라

> In summary, when used sparingly, checked exceptions can increase the reliability of programs; when overused, they make APIs painful to use. If callers won’t be able to recover from failures, throw unchecked exceptions. If recovery may be possible and you want to *force* callers to handle exceptional conditions, first consider returning an optional. Only if this would provide insufficient information in the case of failure should you throw a checked exception.

> 꼭 필요한 곳에만 사용한다면 검사 예외는 프로그램의 안전성을 높여주지만, 남용하면 쓰기 고통스러운 API를 낳는다. API 호출자가 예외 상황에서 복구할 방법이 없다면 비검사 예외를 던지자. 복구가 가능하고 호출자가 그 처리를 해주길 바란다면, 우선 옵셔널을 반환해도 될지 고민하자. 옵셔널만으로는 상황을 처리하기에 충분한 정보를 제공할 수 없을 때만 검사 예외를 던지자.



- 검사 예외를 던지는 메서드는 스트림 안에서 직접 사용할 수 없기 때문에(아이템 45~48) 자바 8부터는 부담이 더욱 커졌다.
- API를 제대로 사용해도 발생할 수 있는 예외, 프로그래머가 의미 있는 조치를 취할 수 있는 경우에는 검사 예외 오케이
  - 그러나 둘 중 어디에도 해당하지 않는다면 비검사 예외를 사용하는 게 좋다. 

```java
// 검사 예외를 잡아서 다시 Error를 던질건가
} catch(TheCheckedException e) {
  throw new AssertinError();
}

// 검사 예외를 잡아서 프린트하고 시스템 종료한다? 
// 이럴거면 차라리 비검사 예외를 선택하자.
} catch(TheCheckedException e) {
  e.printStackTrace();
  System.exit(-1);
}
```



- 이미 다른 검사 예외를 던지는 상황에서 또 다른 검사 예외를 추가하는 경우에는 기껏해야 `catch` 문 하나 추가하는 정도임.
  - **검사 예외가 단 하나뿐이라면** 오직 그 예외 때문에 API 사용자는 try 블록을 추가해야 하고 스트림에서 직접 사용하지 못하게 된다.
    - 이런 상황이라면 검사 예외를 안 던지는 방법이 없는지 고민해볼 가치가 있디.
- 검사 예외를 회피하는 방법은 
  - 가장 쉬운 방법은 **적절한 결과 타입을 담은 옵셔널을 반환** 하는 것이다(아이템 55).
    - 단점은 예외가 발생한 이유를 알려주는 부가 정보를 담을 수 없다는 것.
      - 예외를 사용하면 구체적인 예외 타입과 그 타입이 제공하는 메서드들을 활용해 부가 정보를 제공할 수 있다(아이템 70).
  - 또 다른 방법으로, **검사 예외를** 던지는 메서드를 2개로 쪼개 **비검사 예외로** 바꿀 수 있다.
    - **상태 검사 메서드를 추가**
    - 모든 상황에 적용할 수는 없겠지만 적용할 수만 있다면 **더 아름답지는 않지만 더 쓰기 편하고 유연한 API**를 제공할 수 있다.
    - 코드 71-1 예제
      - actionPermitted()는 상태 검사 메서드에 해당하므로 아이템 69에서 말한 단점도 그대로 적용되니 주의해야 한다.
        - 호출 사이에 객체의 상태가 변할 수 있고 작업 일부를 중복 수행한다는 부분을 애기 하는 것 같다. 
        - 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인에 의해 상태가 변할 수 있다면 코드 71-2 리팩토링은 적절하지 않다.

```java 
// 코드 71-1 검사 예외를 던지는 메서드 - 리팩터링 전
try {
  object.action(args);
} catch(TheCheckedException e) {
  ... // 예외 상황에 대처한다. 
}

// 코드 71-2 상태 검사 메서드와 비검사 예외를 던지는 메서드 - 리팩터링 후
if (obj.actionPermitted(args)) {
	object.action(args);
} else {
  ... // 예외 상황에 대처한다. 
}
```



## 아이템 72. 표준 예외를 사용하라

- 숙련된 프로그래머는 그렇지 못한 프로그래머보다 더 많은 코드를 재사용한다.
  - 예외도 마찬가지로 재사용하는 것이 좋다.
    - 자바 라이브러리는 대부분 API에서 쓰기에 충분한 수의 예외를 제공한다.
- 표준 예외를 재사용하면 얻는 게 많다.
  - 그중 최고는 여러분의 API가 다른 사람이 익히고 사용하기 쉬워진다는 것이다.
  - 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다.



- 많이 재사용되는 예외
  - `IllegalArgumentException`
    - 허용하지 않는 값이 인수로 건네졌을 때(`null`은 따로 `NullPointerException`으로 처리)
  - `IllegalStateException`
    - 객체가 메서드를 수행하기에 적절하지 않은 상태일 때
  - `NullPointerException`
    - `null`을 허용하지 않는 메서드에 `null`을 건넸을 때
  - `IndexOutOfBoundsException`
    - 인덱스가 범위를 넘어섰을 때
  - `ConcurrentModificationException`
    - 허용하지 않는 동시 수정이 발견됐을 때
  - `UnsupportedOperationException`
    - 호출한 메서드를 지원하지 않을 때



- **Exception**, **RuntimeException**, **Throwable**, **Error**는 **직접 재사용하지 말자**.
  - 이 클래스들은 추상 클래스라고 생각하자.
  - 여러 성격의 예외들을 포괄하는 클래스이므로 안정적으로 테스트할 수 없다.
- 상황에 부합한다면 항상 표준 예외를 재사용하자.
  - 이때 API 문서를 참고해 그 예외가 어떤 상황에서 던져지는지 꼭 확인해야 한다.
  - 예외의 이름뿐 아니라 예외가 던져지는 맥락도 부합할 때만 재사용한다.
  - 더 많은 정보를 제공하길 원한다면 표준 예외를 확장해도 좋다.
    - 단, 예외는 직렬화할 수 있다는 사실을 기억하자(12장).
    - 직렬화에는 많은 부담이 따른다는 사실만으로도 나만의 예외를 새로 만들지 않아야 할 근거로 충분할 수 있다.
- 표로 정리한 '주요 쓰임'이 상호 배타적이지 않은 탓에, 종종 재사용 할 예외를 선택하기가 어려울 때도 있다.
  - 카드 덱을 표현하는 객체, 인수로 건넨 수만큼의 카드를 뽑아 나눠주는 메서드
    - 덱에 남아 있는 카드 수보다 큰 값을 건네면 어떤 예외를 던져야 할까?
    - 인수의 값이 너무 크다고 본다면 `IllegalArgumentException`을, 덱에 남은 카드 수가 너무 적다고 본다면 `IllegalStateException`을 선택할 것이다.
      - **이런 상황에서의 일반적인 규칙**
        - 인수 값이 무엇이었든 어차피 실패했을 거라면 `IllegalStateException`을, 그렇지 않으면 `IllegalArgumentException`을 던지자.



## 아이템 73. 추상화 수준에 맞는 예외를 던지라

> In summary, if it isn’t feasible to prevent or to handle exceptions from lower layers, use exception translation, unless the lower-level method happens to guarantee that all of its exceptions are appropriate to the higher level. Chaining provides the best of both worlds: it allows you to throw an appropriate higher-level exception, while capturing the underlying cause for failure analysis (Item 75).

> 아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하라. 이때 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에 좋다(아이템 75).



- **예외 번역(**exception translation)
  - 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다.

```java
// 코드 73-1 예외 번역
try {
  	... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
  	... // 추상화 수준에 맞게 번역한다.
    throw new HigherLevelException(...);  
}
```



- 예외를 밖으로 던지더라도 받는쪽에서 알아먹을 수 있도록 던지자.
  - 안에서 해결할건 하고
  - 안에서 볼 때의 예외와 밖에서 볼 때의 예외는 다르다.

```java
/**
 *  ...
 *
 * @throws IndexOutOfBoundsException {@inheritDoc}
 *
 * NoSuchElementException가 발생하지만 
 * IndexOutOfBoundsException을 던지라고 API에 명시되어 있다.
 */
public E get(int index) {
	ListIterator<E> i = listIterator(index);
  try {
    	return i.next();
  } catch (NoSuchElementException var3) {
    	throw new IndexOutOfBoundsException("인덱스: " + index);
  }
}
```



- 예외를 번역할 때, 저수준 예외가 디버깅에 도움이 된다면 **예외 연쇄**(exception chaining)를 사용하는 게 좋다.
  - 예외 연쇄란 문제의 근본 원인(cause)인 저수준 예외를 고수준 예외에 실어 보내는 방식
  - 예외를 연쇄를 사용하면 별도의 접근자 메서드(`Throwable`의 getCause 메서드)를 통해 필요하면 언제든 저수준 예외를 꺼내 볼 수 있다.
  - 대부분의 표준 예외는 예외 연쇄용 생성자를 갖추고 있다.
    - 그렇지 않은 예외라도 `Throwable`의 initCause 메서드를 이용해 '원인'을 직접 못박을 수 있다.
      - 접근은 getCause 메서드로

```java
// 코드 73-2 예외 연쇄
try {
  	... // 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
  	// 저수준 예외를 고수준 예외에 실어 보낸다.
  	throw new HigherLevelException(cause);
}
```

```java
// 코드 73-3 예외 연쇄용 생성자
class HigherLevelException extends Exception {
  	HigherLevelException(Throwable cause) {
    	super(cause);
  	}
}
```



## 아이템 74. 메서드가 던지는 모든 예외를 문서화하라

> In summary, document every exception that can be thrown by each method that you write. This is true for unchecked as well as checked exceptions, and for abstract as well as concrete methods. This documentation should take the form of @throws tags in doc comments. Declare each checked exception individually in a method’s throws clause, but do not declare unchecked exceptions. If you fail to document the exceptions that your methods can throw, it will be difficult or impossible for others to make effective use of your classes and interfaces.


> 메서드가 던질 가능성이 있는 모든 예외를 문서화하라. 검사 예외든 비검사 예외든, 추상 메서드든 구체 메서드든 모두 마찬가지다. 문서화에는 자바독의 @throws 태그를 사용하면 된다. 검사 예외만 메서드 선언의 throws 문에 일일이 선언하고, 비검사 예외는 메서드 선언에는 기입하지 말자. 발생 가능한 예외를 문서로 남기지 않으면 다른 사람이 그 클래스나 인터페이스를 효과적으로 사용하기 어렵거나 심지어 불가능할 수도 있다.



- 메서드가 던지는 예외는 그 메서드를 올바로 사용하는 데 아주 중요한 정보다.
  - 따라서 각 메서가 던지는 예외 하나하나를 문서화하는 데 충분한 시간을 쏟아야 한다(아이템 56).
- 검사 예외는 항상 따로따로 선언하고, 각 예외가 발생하는 상황을 자바독의 `@throws` 태그를 사용하여 정확히 문서화하자.
  - 공통 상위 클래스 하나로 뭉뚱그려 선언하는 일을 삼가자.
    - 극단적인 예로 메서드가 `Exception`이나 `Throwable`을 던진다고 선언해서는 안 된다.
- 비검사 예외는 일반적으로 프로그래밍 오류를 뜻하는데(아이템 70), 자신이 일으킬 수 있는 오류들이 무엇인지 알려주면 프로그래머는 자연스럽게 해당 오류가 나지 않도록 코딩하게 된다.
  - `public` 메서드라면 필요한 전제조건을 문서화해야 하며(아이템 56), 그 수단으로 가장 좋은 것이 바로 비검사 예외들을 문서화하는 것이다.
- 메서드가 던질 수 있는 예외를 각각 `@throws` 태그로 문서화하되, 비검사 예외는 메서드 선언의 `throws` 목록에 넣지 말자.
  - 검사냐 비검사냐에 따라 API 사용자가 해야 할 일이 달라지므로 이 둘을 확실히 구분해주는 게 좋다.
    - 검사, 비검사 둘 다  `@throws` 태그로 문서화를 한다.
    - 비검사는 당연히 메서드 선언부에 `throws` 뒤에 적어 줄 필요가 없으니 `throws` 뒤에 적혀있는건 검사 예외
      - 검사와 비검사가 시각적으로 구분이 된다. 

- 한 클래스에 정의된 많은 메서드가 같은 이유로 같은 예외를 던진다면 그 예외를 (각각의 메서드가 아닌) 클래스 설명에 추가하는 방법도 있다.
  - `NullPointerException`이 가장 흔한 사례다.
    - 이럴 때는 클래스의 문서화 주석에 "이 클래스의 모든 메서드는 인수로 `null`이 넘어오면 `NullPointerException`을 던진다"라고 적어도 좋다.



## 아이템 75. 예외의 상세 메세지에 실패 관련 정보를 담으라

- 예외를 잡지 못해 프로그램이 실패하면 자바 시스템은 그 예외의 스택 추적(stack trace) 정보를 자동으로 출력한다.
  - 실패 원인을 분석할 때 이 정보가 유일한 정보인 경우가 많다.
    - 실패를 재현하기 어렵다면 더 자세한 정보를 얻기가 어렵거나 불가능하다.
  - 예외의 toString 메서드에 실패 원인에 관한 정보를 가능한 한 많이 담아 반환하는 일은 아주 중요하다.
  - 사후 분석을 위해 실패 순간의 상황을 정확히 포착해 예외의 상세 메세지에 담아야 한다.
- 실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다.
  - 보안과 관련한 정보는 주의해서 다뤄야 한다.
    - 상세 메시지에 비밀번호나 암호 키 같은 정보까지 담아서는 안된다.
  - 관련 데이터를 모두 담아야 하지만 장황할 필요는 없다.
    - 그러니 문서와 소스코드에서 얻을 수 있는 정보는 길게 늘어놔봐야 군더더기가 될 뿐이다.
- 예외의 상세 메세지와 최종 사용자에게 보여줄 오류 메시지를 혼동해서는 안 된다.
  - 최종 사용자에게는 친절한 안내 메시지를 보여줘야 하는 반면, 예외 메시지는 가독성보다는 담긴 내용이 훨씬 중요하다.
    - 예외 메시지의 주 소비층은 문제를 분석해야 할 프로그래머와 SRE 엔지니어임.
- 실패를 적절히 포착하려면 필요한 정보를 **예외 생성자에서** 모두 받아서 **상세 메시지까지 미리 생성**해놓은 방법도 괜찮다.
  - 밑에 예제를 보면 생성자에서 상세 메시지 포맷을 미리 만들어 놓았다.

```java
/**
 * IndexOutOfBoundsException을 생성한다.
 *
 * @param lowerBound 인덱스의 최솟값
 * @param upperBound 인덱스의 최댓값 + 1
 * @param index      인덱스의 실젯값
 */
public IndexOutOfBoundsException(int lowerBound, int upperBound,
                                 int index) {
    // 실패를 포착하는 상세 메시지를 생성한다.
    super(String.format(
            "최솟값: %d, 최댓값: %d, 인덱스: %d",
            lowerBound, upperBound, index));

    // 프로그램에서 이용할 수 있도록 실패 정보를 저장해둔다.
    this.lowerBound = lowerBound;
    this.upperBound = upperBound;
    this.index = index;
}
```



- 'toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자'하는 일반 원칙(75쪽)을 따른다는 관점에서, 비검사 예외라도 상세 정보를 알려주는 접근자 메서드를 제공하라고 권하고 싶다.



## 아이템 76. 가능한 한 실패 원자적으로 만들라

- 일반화해 이야기하면, 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 한다.
  - 이러한 특성을 **실패 원자적**(failure-atomic)이라고 한다.



- 메서드를 실패 원자적으로 만드는 가장 간단한 방법은 **불변 객체**(아이템 17)로 설계하는 것이다.
  - 불변 객체의 상태는 생성 시점에 고정되어 절대 변하지 않기 때문이다.



- 가변 객체의 메서드를 실패 원자적으로 만드는 가장 흔한 방법은 **작업 수행에 앞서 매개변수의 유효성을 검사하는 것**이다(아이템 49).
  - 사실 밑에 예제에서 if문을 제거하더라도 스택이 비었다면 여전히 예외를 던진다.
    - 다만 증감 연산자가 앞에 오면서(--size) size의 값이 음수가 되고 다음번 호출도 실패하게 되고, 이때 던지는 `ArrayIndexOutOfBoundsException`은 추상화 수준이 상황에 어울리지 않는다고 볼 수 있다(아이템 73).
      - `if (size == 0)`으로 거르면 size가 0이 되면 그때부터 pop 메서드가 호출되면 추상화 수준에 맞는 `EmptyStackException`을 던진다.

```java
public Object pop() {
  	if (size == 0)
    		throw new EmptyStackException();
  	Object result = elements[--size];
  	elements[size] = null; // 다 쓴 참조 해제
  	return result;
}
```

- 위와 비슷한 취지로 실패할 가능성이 있는 모든 코드를, 객체의 상태를 바꾸는 코드보다 앞에 배치하는 방법도 있다.
  - `TreeMap`에 엉뚱한 타입의 원소를 추가하려 들면 트리를 변경하기 앞서, 해당 원소가 들어갈 위치를 찾는 과정에서 `ClassCastException`을 던질 것이다.



- 실패 원자성을 얻는 세 번째 방법은 **객체의 임시 복사본에서 작업을 수행한 다음, 작업이 성공적으로 완료되면 원래 객체와 교체하는 것**이다.
  - 데이터를 임시 자료구조에 저장해 작업하는 게 더 빠를 때 적용하기 좋은 방식이다.
    - 정렬 메서드에서는 정렬을 수행하기 전에 입력 리스트의 원소들을 배열로 옮겨 담는다.
      - 배열을 사용하면 정렬 알고리즘의 반복문에서 원소들에 훨씬 빠르게 접근할 수 있기 때문
  - 성능을 높이고자 취한 결정 이지만, 혹시나 정렬에 실패하더라도 입력 리스트는 변하지 않는 효과를 덤으로 얻게 된다.



- 마지막 방법으로 **작업 도중 발생하는 실패를 가로채는 복구 코드를 작성하여 작업 전 상태로 되돌리는 방법**이다.
  - 주로 (디스크 기반의) 내구성(durability)을 보장해야 하는 자료구조에 쓰이는데, 자주 쓰이는 방법은 아니다.



- 실패 원자성은 일반적으로 권장되는 덕목이지만 항상 달성할 수 있는 것은 아니다.
  - 두 스레드가 동기화 없이 같은 객체를 동시에 수정한다면 그 객체의 일관성이 깨질 수 있다.
- 실패 원자적으로 만들 수 있더라도 항상 그리 해야 하는 것도 아니다.
  - 실패 원자성을 달성하기 위한 비용이나 복잡도가 아주 큰 연산도 있기 때문이다.

- **메서드 명세에 기술한 예외라면 설혹 예외가 발생하더라도 객체의 상태는 메서드 호출 전과 똑같이 유지돼야 한다는 것이 기본 규칙이다.**
  - 이 규칙을 지키지 못한다면 실패 시의 객체 상태를 API 설명에 명시해야 한다.
  - 이것이 이상적이나, 아쉽게도 지금의 API 문서 상당 부분이 잘 지키지 않고 있다.



## 아이템 77. 예외를 무시하지 말라

- API 설계자가 메서드 선언에 예외를 명시하는 까닭은, 그 메서드를 사용할 때 적절한 조치를 취해달라고 말하는 것이다.
  - API 설계자의 목소리를 흘려버리지 말자.
  - 안타깝게도 예외를 무시하기란 아주 쉽다.
    - 해당 메서드 호출을 `try` 문으로 감싼 후 `catch` 블록에서 아무 일도 하지 않으면 끝이다.

```java
// catch 블록을 비워두면 예외가 무시된다. 아주 의심스러운 코드다!
try {
  	...
} catch(SomeException e) {
}
```



- 예외를 무시해야 할 때도 있다.
  - 예를 들어 `FileInputStream`을 닫을 때가 그렇다.
  - 예외를 무시하기로 했다면 `catch` 블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 `ignored`로 바꿔놓도록 하자.

```java
try {
  	...
    // 변수 이름은 ignored로  
} catch(SomeException ignored) {
  	// 여기에 왜 예외를 무시하기로 했는지 이유를 주석으로 남기자.
}
```



- 예측할 수 있는 예외 상황이든 프로그래밍 오류든, 빈 `catch` 블록으로 못 본 척 지나치면 그 프로그램은 오류를 내재한 채 동작하게 된다.
  - 그러다 어느 순간 문제의 원인과 아무 상관없는 곳에서 갑자기 죽어버릴 수도 있다.
  - 예외를 적절히 처리하면 오류를 완전히 피할 수도 있다.
  - 무시하지 않고 바깥으로 전파되게만 놔둬도 최소한 디버깅 정보를 남긴 채 프로그램이 신속히 중단되게는 할 수 있다.



