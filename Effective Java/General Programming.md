

## 아이템 57. 지역변수의 범위를 최소화하라

- 지역변수의 유효 범위를 최소로 줄이면 **코드 가독성**과 **유지보수성**이 높아지고 **오류 가능성**은 낮아진다.

- 지역변수의 범위를 줄이는 가장 강력한 기법은 역시 **'가장 처음 쓰일 때 선언하기'**다.
  - 사용하려면 멀었는데, 미리 선언부터 해두면 코드가 어수선해져 가독성이 떨어진다.
  - 변수를 실제로 사용하는 시점엔 타입과 초깃값이 기억나지 않을 수도 있다.
  - 지역변수를 생각 없이 선언하다 보면 변수가 쓰이는 범위보다 너무 앞서 선언하거나, 다 쓴 뒤에도 여전히 살아 있게 되기 쉽다.



- 거의 모든 지역변수는 **선언과 동시에 초기화**해야 한다.
  - 초기화에 필요한 정보가 충분하지 않다면 충분해질 때까지 선언을 미뤄야 한다.
  - `try-catch` 문은 이 규칙에서 예외
    - 변수를 초기화하는 표현식에서 검사 예외를 던질 가능성이 있다면 try 블록 안에서 초기화해야 한다.
      - 그렇지 않으면 예외가 블록을 넘어 메서드에까지 전파된다.
    - 변수 값을 try 블록 바깥에서도 사용해야 한다면 try 블록 앞에서 선언해야 한다.
      - 변수 초기화가 검사 예외를 던질 가능성이 있으면 초기화는 블록 안에서 



- 반복문은 독특한 방식으로 변수 범위를 최소화해준다.
  - 예전의 `for` 형태든 새로운 `for-each` 형태든, 반복문에서는 반복 변수(loop variable)의 범위가 반복문의 몸체, 그리고 `for` 키워드와 몸체 사이의 괄호 안으로 제한된다.
  - 반복 변수의 값을 반복문이 종료된 뒤에도 써야 하는 **상황이 아니라면** `while` 문보다는 `for` 문을 쓰는 편이 낫다.

```java
// 코드 57-1 컬렉션이나 배열을 순회하는 권장 관용구

for (Element e : c) {
  	... // e로 무언가를 한다.
}
```

```java
// 코드 57-2 반복자가 필요할 때(remove 메서드를 써야 한다든가)의 관용구

for (Iterator<Element> i = c.iterator(); i.hasNext();) {
  	Element e = i.next();
  	... // e와 i로 무언가를 한다.
}
```

```java
// 반면 while 문을 쓴다면...
// 반복문이 종료된 뒤에도 반복 변수의 값을 사용할 수는 있다.

Iterator<Element> i = c.iterator();
while (i.hasNext()) {
  	doSomething(i.next());
}
...
  
Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) {								// 버그!
  	// 프로그램 오류가 겉으로 드러나지 않으니 오랜 기간 발견되지 않을 수도 있다.
  	doSomethingElse(i2.next());
}
```

```java
// for 문을 사용하면 위와 같은 복사 붙여넣기 오류를 캄파일 타임에 잡아준다.
// 첫 번째 반복문이 사용한 원소와 반복자의 유효 범위가 반복문 종료와 함께 끝나기 때문이다.

for (Iterator<Element> i = c.iterator(); i.hasNext();) {
  	Element e = i.next();
  	... // e와 i로 무언가를 한다.
}

// 다음 코드는 "i를 찾을 수 없다"는 컴파일 오류를 낸다.
for (Iterator<Element> i2 = c2.iterator(); i.hasNext();) {
  	Element e = i.next();
  	... // e2와 i2로 무언가를 한다.
}
```



- `for` 문이 복사 붙여넣기 오류를 줄여주는 이유는 또 있다. 
  - 변수 유효 범위가 `for` 문 범위와 일치하여 똑같은 이름의 변수를 여러 반복문에서 써도 서로 아무런 영향을 주지 않는다.
  - 사실 이렇게 쓰는 게 더 세련되기까지 하다.
- `for` 문의 장점을 하나만 더 이야기 한다면 `while` 문보다 짧아서 가독성이 좋다는 점이다.



- 밑에 예제의 관용구에서 주목할 부분은 범위가 정확히 일치하는 두 반복 변수 i와 n이다.
  - 반복 여부를 결정짓는 변수 i의 한곗값을 변수 n에 저장하여, 반복 때마다 다시 계산해야 하는 비용을 없앴다.
    - 같은 값을 반환하는 메서드를 매번 호출한다면 이 관용구를 사용하기 바란다.

```java
for (int i = 0, n = expensiveComputation(); i < n; i++) {
  	... // i로 무언가를 한다.
}

for (int i = 0; i < expensiveComputation(); i++) {
  	// 이렇게 하면 반복할 때마다 expensiveComputation()을 호출한다.
  	...
}
```



- 지역변수 범위를 최소화하는 마지막 방법은 **메서드를 작게 유지하고 한 가지 기능에 집중**하는 것이다.
  - 한 메서드에서 여러 가지 기능을 처리한다면 그중 한 기능과만 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근할 수 있을 것이다.
  - 해결책은 간단하다. 단순히 메서드를 기능별로 쪼개면 된다.



## 아이템 58. 전통적인 for 문 보다는 for-each 문을 사용하라

> In summary, the for-each loop provides compelling advantages over the traditional for loop in clarity, flexibility, and bug prevention, with no performance penalty. Use for-each loops in preference to for loops wherever you can.

> 전통적인 for 문과 비교했을 때 for-each 문은 명료하고, 유연하고, 버그를 예방해준다. 성능 저하도 없다. 가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자.  



- 아래의 관용구들은 `while` 문보다는 낫지만(아이템 57) 가장 좋은 방법은 아니다.
  - 반복자와 인덱스 변수는 모두 코드를 지저분하게 할 뿐 우리에게 진짜 필요한 건 원소들뿐이다.
  - 더군다나 이처럼 쓰이는 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다.
  - 컬렉션이냐 배열이냐에 따라 코드 형태가 상당히 달라지므로 주의해야 한다.

```java
// 코드 58-1 컬렉션 순회하기 - 더 나은 방법이 있다.

for(Iterator<Element> i = c.iterator(); i.hasNext(); ){
    Element e = i.next();
    ... //e로 무언가를 한다.
}
```

```java
// 코드 58-2 배열 순회하기 - 더 나은 방법이 있다.

for(int i = 0 ; i < a.length ; i ++){
    ... // a[i]로 무언가를 한다.
}
```



- 위에서 언급한 문제는 `for-each` 문을 사용하면 모두 해결된다.
  - 참고로 `for-each` 문의 정식 이름은 '향상된 `for` 문(enhanced for statement)'이다.
  - 반복자와 인덱스 변수를 사용하지 않으니 코드가 깔끔해지고 오류가 날 일도 없다.
  - 하나의 과용구로 컬렉션과 배열을 모두 처리할 수 있어서 어떤 컨테이너를 다루는지는 신경 쓰지 않아도 된다.
    - 반복 대상이 컬렉션이든 배열이든, `for-each` 문을 사용해도 속도는 그대로다.

```java
// 코드 58-3 컬렉션과 배열을 순회하는 올바른 관용구
// 이 반복문은 "elements 안의 각 원소 e에 대해"라고 읽는다.

for(Element e : elements){
    ... // e로 무언가를 한다.
}
```



- 컬렉션을 중첩해 순회해야 한다면 `for-each` 문의 이점이 더욱 커진다.
  - 밑에 코드에서 버그를 찾아보자.
    - 여기서 문제는 바깥 컬렉션(suits)의 반복자에서 next 메서드가 너무 많이 불린다는 것이다.
      - i.next()는 '숫자(Suit) 하나당' 한 번씩만 불려야 하는데, 안쪽 반복문에서 호출되는 바람에 '카드(Rank) 하나당' 한 번씩 불리고 있다. 
        - 그래서 숫자가 바닥나면 반복문에서 `NoSuchElementException`을 던진다.

```java
// 코드 58-4 버그를 찾아보자.

enum Suit {CLUB, DIAMOND, HEART, SPADE}
enum RANK { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}

...

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());
 
 
List<Card> dec = new ArrayList<>();
for(Iterator<Suit> i = suits.iterator() ; i.hasNext(); )
    for(Iterator<Rank> j = ranks.iterator(); j.hasNext();)
        dec.add(new Card(i.next(), j.next()));
```



- 주사위를 두 번 굴렸을 때 나올 수 있는 모든 경우의 수를 출력하는 코드
  - 코드 58-5는 예외를 던지진 않지만, 가능한 조합을 단 여섯 쌍만 출력하고 끝나버린다(36개 조합이 나와야 한다).
    - 첫번째 `for`문은 한번 돌고 끝나버린다. 두번째 for문에서 i.next()를 6번 다해 버리니깐.
  - 코드 58-6는 코드 58-5의 문제를 고쳤다.
    - 바깥 반복문에 바깥 원소를 저장하는 변수를 하나 추가
  - 코드 58-7는 `for-each` 문을 중첩해서 코드를 간결하게 만들었다.

```java
// 코드 58-5 같은 버그, 다른 증상!

enum face { ONE, TWO, THREE, FOUR, FIVE, SIX }

...

Collection<Face> faces = EnumSet.allOf(Face.class);
  
for(Iterator<Face> i = faces.iterator(); i.hasNext();)
    for(Iterator<Face> j = faces.iterator(); j.hasNext(); )
        System.out.println(i.next() + " " + j.next());
```

```java
// 코드 58-6 문제는 고쳤지만 보기 좋진 않다. 더 나은 방법이 있다.

for(Iterator<Face> i = faces.iterator(); i.hasNext();) {
  	Suit suit = i.next();	
  
    for(Iterator<Face> j = faces.iterator(); j.hasNext(); )
    		dec.add(new Card(suit, j.next())); 
}
```

```java
// 코드 58-7 컬렉션이나 배열의 중첩 반복을 위한 권장 관용구

for(Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank));
```



- 안타깝게도 `for-each` 문을 **사용할 수 없는 상황이 세 가지** 존재한다.
  - 파괴적인 필터링(destructive filtering)
    - 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다.
    - 자바 8부터는 `Collection`의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.
  - 변형(transforming)
    - 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.
  - 병렬 반복(parallel iteration)
    - 여러 컬렉션을 병렬로 순회해야 한다면 각자의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다(의도한 것은 아니지만 앞서의 코드 58-4가 이러한 사례에 속한다).
- 세 가지 상황 중 하나에 속할 때는 일반적인 `for` 문을 사용하되 이번 아이템에서 언급한 문제들을 경계하기 바란다.



- `for-each` 문은 컬렉션과 배열은 물론 `Iterable` 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.
  - `Iterable` 인터페이스는 다음과 같이 메서드가 단 하나뿐이다.

```java
Public interface Iterable<E>{
		// 이 객체의 원소들을 순회하는 반복자를 반환한다. 		
		Iterator<E> iterator();
}
```



## 아이템 59. 라이브러리를 익히고 사용하라

> To summarize, don’t reinvent the wheel. If you need to do something that seems like it should be reasonably common, there may already be a facility in the libraries that does what you want. If there is, use it; if you don’t know, check. Generally speaking, library code is likely to be better than code that you’d write yourself and is likely to improve over time. This is no reflection on your abilities as a programmer. Economies of scale dictate that library code receives far more attention than most developers could afford to devote to the same functionality.

> 바퀴를 다시 발명하지 말자. 아주 특별한 나만의 기능이 아니라면 누군가 이미 라이브러리 형태로 구현해놓았을 가능성이 크다. 그런 라이브러리가 있다면, 쓰면 된다. 있는지 잘 모르겠다면 찾아보라. 일반적으로 라이브러리의 코드는 여러분이 직접 작성한 것보다 품질이 좋고, 점차 개선될 가능성이 크다. 여러분의 실력을 폄하하는 게 아니라. 코드 품질에도 규모의 경제가 적용된다. 즉, 라이브러리 코드는 개발자 각자가 작성하는 것보다 주목을 훨씬 많이 받으므로 코드 품질도 그만큼 높아진다.



- 표준 라이브러리를 쓰면 얻는 이점
  - 첫 번째 그 코드를 작성한 전문가의 지식과 여러분보다 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다.
    - 자바 7부터는 `Random`을 더 이상 사용하지 않는게 좋다.
      - `ThreadLocalRandom`으로 대체하면 대부분 잘 작동한다.
      - 포크-조인 풀이나 병렬 스트림에서는 `SplittableRandom`
  - 두 번째 이점은 핵심적인 일과 크게 관련 없는 문제를 해결하느라 시간을 허비하지 않아도 된다는 것이다.
  - 세 번째 이점은 따로 노력하지 않아도 성능이 지속해서 개선된다는 점이다.
  - 네 번째 이점은 기능이 점점 많아진다는 것이다.
  - 마지막 이점은 여러분이 작성한 코드가 많은 사람에게 낯익은 코드가 된다는 것이다.
    - 자연스럽게 다른 개발자들이 더 읽기 좋고, 유지보수하기 좋고, 재활용하기 쉬운 코드가 된다.



- 표준 라이브러리에 그런 기능이 있는지를 모르고 프로그래머가 직접 구현해 쓰는 경우가 많다.
  - 메이저 릴리스마다 주목할 만한 수많은 기능이 라이브러리에 추가된다.
    - 그 예로 자바 9에서 `InputStream`에 추가된 `transferTo` 메서드를 사용하면 쉽게 지정한 URL의 내용을 가져올 수 있다.
  - 라이브러리가 너무 방대하여 모든 API 문서를 공부하기는 벅차겠지만 자바 프로그래머라면 적어도 `java.lang`, `java.util`, `java.io`와 그 하위 패키지들에는 익숙해져야 한다.
  - 언급할 만한 라이브러리로는 컬렉션 프레임워크, 스트림 라이브러리(아이템 45~48), `java.util.concurrent`(아이템 80~81) 등등 
  - 어떤 라이브러리든 제공하는 기능은 유한하므로 항상 빈 구멍이 있기 마련이다. 자바 표준 라이브러리에서 원하는 기능을 찾지 못하면, 그다음 선택지는 고품질의 서드파티 라이브러리가 될 것이다.
    - 대표적으로 구글의 구아바 라이브러리



## 아이템 60. 정확한 답이 필요하다면 float와 double은 피하라

> In summary, don’t use float or double for any calculations that require an exact answer. Use BigDecimal if you want the system to keep track of the decimal point and you don’t mind the inconvenience and cost of not using a primitive type. Using BigDecimal has the added advantage that it gives you full control over rounding, letting you select from eight rounding modes whenever an operation that entails rounding is performed. This comes in handy if you’re performing business calculations with legally mandated rounding behavior. If performance is of the essence, you don’t mind keeping track of the decimal point yourself, and the quantities aren’t too big, use int or long. If the quantities don’t exceed nine decimal digits, you can use int; if they don’t exceed eighteen digits, you can use long. If the quantities might exceed eighteen digits, use BigDecimal.

> 정확한 답이 필요한 계산에는 float나 double을 피하라. 소수점 추적은 시스템에 맡기고, 코딩 시의 불편함이나 성능 저하를 신경 쓰지 않겠다면 BigDecimal을 사용하라. BigDecimal이 제공하는 여덟 가지 반올림 모드를 이용하여 반올림을 완벽히 제어할 수 있다. 법으로 정해진 반올림을 수행해야 하는 비즈니스 계산에서 아주 편리한 기능이다. 반면, 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라. 숫자를 아홉 자리 십진수로 표현할 수 있다면 int를 사용하고, 열여덟 자리 십진수로 표현할 수 있다면 long을 사용하라. 열여덟 자리를 넘어가면 BigDecimal을 사용해야 한다.



- `float`와 `double` 타입은 특히 금융 관련 계산과는 맞지 않는다.
  - `float`와 `double` 타입은 과학과 공학 계산용으로 설계되었다.
    - 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 **'근사치'**로 계산하도록 세심하게 설계되었다.
      - 정확한 결과가 필요할 때는 사용하면 안된다.

```java
System.out.println(1.03 - 0.42); // 0.610000000000000001
```

- 부동소수점
  - 소수점의 위치를 고정하지 않고 그 위치를 나타내는 수를 따로 적는 것
  - double 
    - 부호(1비트) + 지수부(11비트) + 가수부(52비트) = 64비트
    - 지수부가 십에 몇승
    - double MAX_VALUE
      - 1.7976931348623157e+308 or 1.7976931348623157e308
        - e308
        - 10에 308승까지 가능하다는 소리다.
      - `double a = 999~9.0;` 9를 백게 double에 넣고 콘솔에 찍으면
        - 1.0E100



- 금융 계산에 부동소수 타입을 사용해서 오류가 발생하는 코드

```java
double funds = 1.00;
int itemsBought = 0;
for (double price = 0.10; funds >= price; price += 0.10) {
  funds -= price;
  itemsBought++;
}
System.out.println(itemsBought + "개 구입");
System.out.println("잔돈(달러): " + funds);
```



- `BigDecimal`을 사용한 해법
  - 속도가 느리고 쓰기 불편하다. 

```java
final BigDecimal TEN_CENTS = new BigDecimal(".10");

int itemsBought = 0;
BigDecimal funds = new BigDecimal("1.00");
for (BigDecimal price = TEN_CENTS;
     funds.compareTo(price) >= 0;
     price = price.add(TEN_CENTS)) {
  
    funds = funds.subtract(price);
    itemsBought++;
}
System.out.println(itemsBought + "개 구입");
System.out.println("잔돈(달러): " + funds);
```



- 정수 타입을 사용한 해법
  - `int`, `long` 타입을 쓰면 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야 한다.
    - 예제에서는 모든 계산을 달러 대신 센트로 수행해서 이 문제가 해결했다.

```java
int itemsBought = 0;
int funds = 100;
for (int price = 10; funds >= price; price += 10) {
  funds -= price;
  itemsBought++;
}
System.out.println(itemsBought + "개 구입");
System.out.println("잔돈(센트): " + funds);
```



## 아이템 61. 박싱된 기본 타입보다는 기본 타입을 사용하라

> In summary, use primitives in preference to boxed primitives whenever you have the choice. Primitive types are simpler and faster. If you must use boxed primitives, be careful! Autoboxing reduces the verbosity, but not the danger, of using boxed primitives. When your program compares two boxed primitives with the == operator, it does an identity comparison, which is almost certainly *not* what you want. When your program does mixed-type computations involving boxed and unboxed primitives, it does unboxing, and when your program does unboxing, it can throw a NullPointerException. Finally, when your program boxes primitive values, it can result in costly and unnecessary object creations.

> 기본 타입과 박싱된 기본 타입 중 하나를 선택해야 한다면 가능하면 기본 타입을 사용하라. 기본 타입은 간단하고 빠르다. 박싱된 기본 타입을 써야 한다면 주의를 기울이자. 오토박싱이 박싱된 기본 타입을 사용할 때의 번거로움을 줄여주지만, 그 위험까지 없애주지는 않는다. 두 박싱된 기본 타입을 == 연산자로 비교한다면 식별성 비교가 이뤄지는데, 이는 여러분이 원한 게 아닐 가능성이 크다. 같은 연산에서 기본 타입과 박싱된 기본 타입을 혼용하면 언박싱이 이뤄지며, 언박싱 과정에서 NullPointerException을 던질 수 있다. 마지막으로, 기본 타입을 박싱하는 작업은 필요 없는 객체를 생성하는 부작용을 나을 수 있다.



### 자바 데이터 타입

- 기본 타입
  - `int`, `double`, `boolean`
  - 각각의 기본 타입에는 대응하는 참조 타입이 하나씩 있다.
    - 박싱된 기본 타입
      - `Integer`, `Double`, `Boolean`
- 참조 타입
  - `String`, `List`



### 기본 타입과 박싱된 기본 타입 비교

- 기본 타입과 박싱된 기본 타입의 주된 차이
  - 기본 타입은 값만 가지고 있는데 박싱된 기본 타입은 값에 대해서 **식별성**이라는 속성을 가진다.
    - 값이 같아도 서로 다르다고 식별 될 수 있다.
  - 기본 타입의 값은 언제나 유효하다. 
    - 하지만 박싱된 기본 타입은 유효하지 않은 값(`null`) 을 가질 수 있다.
  - 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.




- 박싱된 기본 타입에 **== 연산자**를 사용하면 오류가 일어난다.
  - 지역변수 2개를 두어 매개변수의 값을 기본 타입 정수로 저장하고 **모든 비교를 이 기본 타입 변수로 수행**

```java
// 코드 61-1 잘못 구현된 비교자 - 문제를 찾아보자!
public class BrokenComparator {
    public static void main(String[] args) {

// 				두 '객체 참조'의 식별성을 검사 (i == j)     
//        Comparator<Integer> naturalOrder =
//                (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);

       // 코드 61-2 문제를 수정한 비교자 (359쪽)
        Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
            int i = iBoxed, j = jBoxed; // 오토박싱
            return i < j ? -1 : (i == j ? 0 : 1);
        };

        int result = naturalOrder.compare(new Integer(42), new Integer(42));
        System.out.println(result);
    }
}
```



- **기본 타입과 박싱된 기본 타입을 혼용한 연산**에서는 박싱된 기본 타입의 박싱이 자동으로 풀린다.
  - null 참조를 언박싱하면 NullPointerException이 발생한다.

```java
// 코드 61-3 기이하게 동작하는 프로그램 - 결과를 맞혀보자!
public class Unbelievable {
//    static Integer i;
    static int i;

    public static void main(String[] args) {
        if (i == 42)
            System.out.println("믿을 수 없군!");
    }
}
```



- 지역변수 sum을 박싱된 기본 타입으로 선언하여 느려졌다.
  - 오류나 경고 없이 컴파일 되지만, 박싱과 언박싱이 반복해서 일어나 **성능**이 졸라 느려졌다.

```java
// 끔찍이 느리다!
public class MaxValue {
    public static void main(String[] args) {
        LocalDateTime start = LocalDateTime.now();
//        Long sum = 0L; // 6초
        long sum = 0L; // 0초

        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            sum += i;
        }
        LocalDateTime end = LocalDateTime.now();
        System.out.println(Duration.between(start, end).getSeconds());
        System.out.println(sum);
    }
}
```



### 박싱된 기본 타입 언제 써야 하나

- 박싱된 기본 타입을 컬렉션의 원소, 키, 값으로 쓴다.
  - 컬렉션은 기본 타입을 담을 수 없으므로 어쩔 수 없이 박싱된 기본 타입을 써야만 한다.

- 매개변수화 타입(`List<String>`) 이나 매개변수화 메서드(5장)의 타입 매개변수로는 박싱된 기본 타입을 써야 한다.

  - 자바 언어가 타입 매개변수로 기본 타입을 지원하지 않기 때문이다.

  - `ThreadLocal<int>`타입으로 선언은 불가능하고 `ThreadLocal<Integer>`로 선언해야 한다.

- 리플렉션을 통해 메서드를 호출할 때도 박싱된 기본 타입을 사용해야 한다.



## 아이템 62. 다른 타입이 적절하다면 문자열 사용을 피하라

> To summarize, avoid the natural tendency to represent objects as strings when better data types exist or can be written. Used inappropriately, strings are more cumbersome, less flexible, slower, and more error-prone than other types. Types for which strings are commonly misused include primitive types, enums, and aggregate types.

> 더 적합한 데이터 타입이 있거나 새로 작성할 수 있다면, 문자열을 쓰고 싶은 유혹을 뿌리쳐라. 문자열은 잘못 사용하면 번거롭고, 덜 유연하고, 느리고, 오류 가능성도 크다. 문자열을 잘못 사용하는 흔한 예로는 기본 타입, 열거 타입, 혼합 타입이 있다.



### 문자열을 쓰지 않아야 할 사례

- 문자열은 다른 값 타입을 대신하기에 적합하지 않다.
  - 기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고 없다면 새로 하나 작성해라.
- 문자열은 열거 타입을 대신하기에 적합하지 않다.
- 문자열은 혼합 타입을 대신하기에 적합하지 않다.
  - 적절한 `equals`, `toString`, `compareTo` 메서드를 제공할 수 없으며, `String`이 제공하는 기능에만 의존해야 한다.
  - 차라리 **전용 클래스를 새로** 만드는 편이 낫고 보통 이런 클래스는 **`private` 정적 멤버 클래스로 선언**한다(아이템 24).

```java
// 코드 62-1 혼합 타입을 문자열로 처리한 부적절한 예
String compundKey = className + “#” + i.next();
```



### 문자열은 권한을 표현하기에 적합하지 않다.

- 자바 2 이전에는 권한(capacity)을 문자열로 표현하는 경우가 종종 있다.	
  - 각 스레드가 자신만의 변수를 갖게 해주는 기능
    - 클라이언트가 제공한 문자열 키로 스레드별 지역변수를 식별
  - 이 방식의 문제는 스레드 구분용 문자열 키가 전역 이름공간에서 공유 된다는 점
    - 두 클라이언트가 서로 소통이 안되서 각 클라이언트의 키가 고유하지 않다면 둘 다 제대로 기능하지 못 할 것이다.
    - 보안에도 취약해서 악의적 의도를 가지고 같은 키를 사용하여 다른 클라이언트의 값을 가져올 수도 있다. 

```java
// 코드 62-2 잘못된 예 - 문자열을 사용해 권한을 구분하였다.
public class ThreadLocal {
    private ThreadLocal() { } // 객체를 만들 수 없다.

    // 주어진 이름이 가리키는 스레드 지역 변수의 값 설정
    public static void set(String key, Object value);

    // 주어진 이름이 가리키는 스레드 지역 변수의 값 반환
    public static Object get(String key);
}
```



- 이 API는 문자열 대신 위조할 수 없는 키를 사용하면 해결된다. 이 키를 권한(capacity)이라고도 한다.
  - 스레드 지역 변수를 구분하기 위한 키를 `String` 문자열에서 `Key` 클래스로

```java
// 코드 62-3 Key 클래스로 권한을 구분했다.
public class ThreadLocal {
    private ThreadLocal() { } // 객체를 만들 수 없다.

    public static class Key { // 권한
        Key() { }
    }

    // 유일성이 보장되는, 위조 불가능 키를 생성
    public static Key getKey {
        return new Key();
    }

    public static void set(Key key, Object value);
    public static Object get(Key key); 
}
```



- get(), set()을 `Key` 클래스 안으로 옮기고 별달리 하는 일이 없어진 `ThreadLocal` 클래스는 치워버린다. 
  - `Key` 클래스 이름을 `ThreadLocal`로 변경.
  - set() 메소드를 통해 스레드가 변수에 담고 싶은 값을 넣으면 된다.
  - `java.lang.ThreadLocal`과 흡사해졌다.

```java
// 코드 62-4 리팩터링하여 Key를 ThreadLocal로 변경
public final class ThreadLocal {
    public ThreadLocal();
    public void set(Object value);
    public Object get();
}
```

```java
// 코드 62-5 매개변수화하여 타입안정성 확보
public final class ThreadLocal<T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
```

```java
// java.lang.ThreadLocal
public class ThreadLocal<T> {
		private final int threadLocalHashCode = nextHashCode();
  	public ThreadLocal() {}
  	private T setInitialValue() {...}
  	public T get() {...}
}  
```



## 아이템 63. 문자열 연결은 느리니 주의하라

> The moral is simple: Don’t use the string concatenation operator to combine more than a few strings unless performance is irrelevant. UseStringBuilder’s append method instead. Alternatively, use a character array, or process the strings one at a time instead of combining them.

> 원칙은 간단하다. 성능에 신경 써야 한다면 많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하자. 대신 StringBuilder의 append 메서드를 사용하라. 문자 배열을 사용하거나, 문자열을 (연결하지 않고) 하나씩 처리하는 방법도 있다.



### 문자열 연결 연산자(+)

- 여러 문자열을 하나로 합쳐주는 편리한 수단
- 한 줄짜리 출력값 혹은 작고 크기가 고정된 객체의 문자열 표현은 오케이
- 본격적으로 사용하기 시작하면 성능 저하
  - 문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다.
  - 문자열은 불변이라서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야한다.

```java
// 코드 63-1 문자열 연결을 잘못 사용한 예 - 느리다!
public String statement() {
    String result = "";
    
  	for (int i =0 ; i < numItems(); i++){
        result += lineForItem(i); // 문자열 연결
    }
  	return result;
}
```



### StringBuilder

- 성능을 포기하고 싶지 않다면 `String` 대신 `StringBuilder`를 사용하자.
  - statement 메소드의 수행 시간은 품목 수의 제곱에 비례해 늘어난다.
  - statement2 메소드의 수행 시간은 품목 수의 제곱이 선형으로 늘어난다.
- `StringBuilder`를 전체 결과를 담기에 충분한 크기로 초기화한 점을 잊지 말자.
  - 기본값을 사용해도 훨씬 빠르다.

```java
// 코드 63-2 StringBuilder를 사용하면 문자열 연결 성능이 크게 개선된다.
public String statement2() {
    StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    
  	for (int i =0; i < numItems(); i++) {
        b.append(lineForItem(i));
    }  
    return b.toString();
}
```



## 아이템 64. 객체는 인터페이스를 사용해 참조하라

- 아이템 51에서 매개변수 타입으로 클래스가 아니라 인터페이스를 사용하라고 했다.
  - 이 조언을 "객체는 클래스가 아닌 인터페이스로 참조하라"고까지 확장할 수 있다.
    - 적합한 인터페이스만 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 **전부 인터페이스 타입으로 선언**하라.
    - 객체의 실제 클래스를 사용해야 할 상황은 '오직' 생성자로 생성할 때 뿐이다.

```java
// 좋은 예. 인터페이스를 타입으로 사용했다.
Set<Son> sonSet = new LinkedHashSet<>();

// 나쁜 예. 클래스를 타입으로 사용했다.
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```



- 인터페이스를 타입으로 사용하는 습관을 길러두면 **프로그램이 훨씬 유연**해진다.

```java
// 좋은 예. 인터페이스를 타입으로 사용했다.
Set<Son> sonSet = new LinkedHashSet<>();

// 구현 클래스를 교체하려면 새 클래스의 생성자(혹은 다른 정적 팩터리)를 호출
Set<Son> sonSet = new HashSet<>();
```



- 원래의 클래스가 인터페이스의 일반 규약 이외의 특별한 기능을 제공하며, 주변 코드가 이 기능에 기대어 동작한다면 새로운 클래스도 반드시 같은 기능을 제공해야 한다.
  - 구현 클래스를 `LinkedHashSet`에서 `HashSet`으로 바꿀 경우 `LinkedHashSet`의 순서 정책에 따라 동작하는 코드에서는 문제가 발생 할 수 있다.



- 적합한 인터페이스가 없다면 당연히 클래스로 참조해야 한다.
  - `String`과 `BigInteger` 같은 값 클래스
    - 값 클래스를 여러 가지로 구현될 수 있다고 생각하고 설계하는 일은 거의 없다.
    - `final`인 경우가 많고 상응하는 인터페이스가 별도로 존재하는 경우가 드물다.
  - 클래스 기반으로 작성된 프레임워크가 제공하는 객체
    - 이런 경우는 특정 구현 클래스보다는 기반 클래스를 사용해 참조하는게 좋다.
    - `OutputStream` 등 `java.io` 패키지의 여러 클래스가 이 부류에 속한다.
  - 인터페이스에는 없는 특별한 메서드를 제공하는 클래스
    - `PriorityQueue` 클래스는 `Queue` 인터페이스에는 없는 `comparator` 메서드를 제공한다.
      - 클래스 타입을 직접 사용하는 경우는 이런 추가 메서드를 꼭 사용해야 하는 경우로 최소화해야 하며, 절대 남발하지 말아야 한다.



- 실전에서는 주어진 객체를 표현할 적절한 인터페이스가 있는지 찾아서 그 인터페이스로 참조하면 더 유연하고 세련된 프로그램을 만들 수 있다.
  - 적합한 인터페이스가 없다면 클래스의 계층구조 중 필요한 기능을 만족하는 **가장 덜 구체적인(상위의) 클래스를 타입으로** 사용하자.



## 아이템 65. 리플렉션보다는 인터페이스를 사용하라

> In summary, reflection is a powerful facility that is required for certain sophisticated system programming tasks, but it has many disadvantages. If you are writing a program that has to work with classes unknown at compile time, you should, if at all possible, use reflection only to instantiate objects, and access the objects using some interface or superclass that is known at compile time.

> 리플렉션은 복잡한 특수 시스템을 개발할 때 필요한 강력한 기능이지만, 단점도 많다. 컴파일타임에는 알 수 없는 클래스를 사용하는 프로그램을 작성한다면 리플렉션을 하용해야 할 것이다. 단, 되도록 객체 생성에만 사용하고, 생성한 객체를 이용할 때는 적절한 인터페이스나 컴파일타임에 알 수 있는 상위 클래스를 형변환해 사용해야 한다.



### java.lang.reflect

- 리플렉션 기능(`java.lang.reflect`) 을 이용하면 프로그램에서 임의의 클래스에 접근 할 수 있다.
- **`Class` 객체**가 주어지면 그 클래스의 생성자, 메서드, 필드에 해당하는 **`Constructor`, `Method`, `Field` 인스턴스**를 가져올 수 있고, 이어서 이 인스턴스들로는 그 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있다.
  - `Constructor`, `Method`, `Field` 인스턴스를 통해 해당 클래스의 인스턴스를 생성하거나, 메서드를 호출하거나, 필드에 접근할 수 있다.
  - `Method.invoke`는 어떤 클래스의 어떤 객체가 가진 어떤 메서드라도 호출할 수 있게 해준다.
  - `Class` 객체가 주어지면?
    - 자바 클래스 로더 시스템의 로딩이 끝나면 해당 클래스 타입의 `Class` 객체를 생성하여 힙 영역에 저장



### 리플렉션 단점

- 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다.

  - 개발 시 리플렉션 기능을 써서 존재하지 않거나 접근할 수 없는 메서드를 호출하고 그런지도 모르고 프로그램을 올려서 런타임 오류가 발생할 여지가 있다.

- 리플렉션을 이용하면 코드가 지저분하고 장황해진다.

  - 지루한 일이고, 읽기도 어렵다.

- 성능이 떨어진다.

  - 리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 훨씬 느리다.

  - 저자가 본인의 컴퓨터에서 입력 매개변수가 없고 `int`를 반환하는 메서드로 실험해 보니 11배나 느렸다고 한다.

- 단점이 명벽하기에 코드 분석 도구나 의존관계 주입 프레임워크처럼 리플렉션을 써야 하는 복잡한 애플리케이션에서도 리플렉션 사용을 점차 줄이고 있다.

  - 스프링부트 프록시 생성 디폴트도 **바이트코드 조작**을 사용하는 `CGlib`이다. 



### 리플렉션으로 생성하고 인터페이스로 참조

- 리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취할 수 있다.
- 컴파일타임에 이용할 수 없는 클래스를 사용해야만 하는 프로그램은 비록 컴파일타임이라도 적절한 인터페이스나 상위 클래스를 이용할 수는 있을 것이다(아이템 64).
  - 리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.
- 예제
  - 인텔리제이에서 Program arguments에 데이터 넣을때
    - `java.util.HashSet` 가나다라(O)
    - `java.util.HashSet`**,** 가나다라(X)
  - `getDeclaredConstructor()` 메소드는 `public`, `protected`, `private`, `default` 상관없이 `class` 안의 모든 생성자에 접근 가능
  - 밑에 예제는 리플렉션의 단점 두 가지를 보여준다.
    - 첫 번째, 런타임에 총 여섯 가지나 되는 예외를 던질 수 있다.
      - 그 모두가 인스턴스를 리플렉션 없이 생성했다면 컴파일타임에 잡아낼 수 있었을 예외들이다.
    - 두 번째, 클래스 이름만으로 인스턴스를 생성해내기 위해 무려 25줄이나 되는 코드를 작성했다.
      - 리플렉션이 아니라면 생성자 호출 한 줄로 끝났을 일이다.
    - 두 단점 모두 객체를 생성하는 부분에만 국한된다.
      - 객체가 일단 만들어지면 그 후의 코드는 여타의 Set 인스턴스를 사용 할 때와 똑같다.

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.util.Arrays;
import java.util.Set;

// 리플렉션으로 활용한 인스턴스화 데모
public class ReflectiveInstantiation {
    // 코드 65-1 리플렉션으로 생성하고 인터페이스로 참조해 활용한다.
    public static void main(String[] args) {
        // 클래스 이름을 Class 객체로 변환
        Class<? extends Set<String>> cl = null;
        try {
            cl = (Class<? extends Set<String>>)  // 비검사 형변환!
                    Class.forName(args[0]);
        } catch (ClassNotFoundException e) {
            fatalError("클래스를 찾을 수 없습니다.");
        }

        // 생성자를 얻는다.
        Constructor<? extends Set<String>> cons = null;
        try {
            cons = cl.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
        }

        // 집합의 인스턴스를 만든다.
        Set<String> s = null;
        try {
            s = cons.newInstance();
        } catch (IllegalAccessException e) {
            fatalError("생성자에 접근할 수 없습니다.");
        } catch (InstantiationException e) {
            fatalError("클래스를 인스턴스화할 수 없습니다.");
        } catch (InvocationTargetException e) {
            fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
        } catch (ClassCastException e) {
            fatalError("Set을 구현하지 않은 클래스입니다.");
        }

        // 생성한 집합을 사용한다.
        s.addAll(Arrays.asList(args).subList(1, args.length));
        System.out.println(s);
    }

    private static void fatalError(String msg) {
        System.err.println(msg);
        System.exit(1);
    }
}
```



## 아이템 66. 네이티브 메서드는 신중히 사용하라

> In summary, think twice before using native methods. It is rare that you need to use them for improved performance. If you must use native methods to access low-level resources or native libraries, use as little native code as possible and test it thoroughly. A single bug in the native code can corrupt your entire application.

> 네이티브 메서드를 사용하려거든 한번 더 생각하라. 네이티브 메서드가 성능을 개선해주는 일은 많지 않다. 저수준 자원이나 네이티브 라이브러리를 사용해야만 해서 어쩔 수 없더라도 네이티브 코드는 최소한만 사용하고 철저히 테스트하라. 네이티브 코드 안에 숨은 단 하나의 버그가 여러분 애플리케이션 전체를 훼손할 수도 있다.



- 자바 네이티브 인터페이스(Java Native Interface, `JNI`)는 자바 프로그램이 네이티브 메서드를 호출하는 기술이다.
  - 여기서 네이티브 메서드란 C나 C++ 같은 네이티브 프로그래밍 언어로 작성한 메서드를 말한다.
- 전통적으로 네이티브 메서드의 주요 쓰임은 다음 세 가지다.
  - 첫 번째, 레지스트리 같은 플랫폼 특화 기능을 사용한다.
  - 두 전째, 네이티브 코드로 작성된 기존 라이브러리를 사용한다.
  - 세 번째, 성능 개선을 목적으로 성능에 결정적인 영향을 주는 영역만 따로 네이티브 언어로 작성한다.
- 성능을 개선할 목적으로 네이티브 메서드를 사용하는 것은 거의 권장하지 않는다.
  - `BigInteger` 에 경우 자바 3 때 순수 자바로 다시 구현되면서 세심히 튜닝한 결과, 원래의 네이티브 구현보다도 더 빨라졌다.
- 네이티브 메서드에는 심각한 단점이 있다.
  - 네이티브 언어가 안전하지 않으므로(아이템 50) 네이티브 메서드를 사용하는 애플리케이션도 메모리 훼손 오류로부터 더 이상 안전하지 않다.
  - 네이티브 언어는 자바보다 플랫폼을 많이 타서 이식성도 낮다.
  - 디버깅도 더 어렵다.
  - 주의하지 않으면 속도가 오히려 느려질 수도 있다.
  - 가지비 컬렉터가 네이티브 메모리는 자동 회수하지 못하고, 심지어 추적조차 할 수 없다(아이템 8).
  - 자바 코드와 네이티브 코드의 경계를 넘나들 때마다 비용도 추가된다.
  - 네이티브 메서드와 자바 코드 사이의 '접착 코드(glue code)'를 작성해야 하는데, 이는 귀찮은 작업이기도 하거니와 가독성도 떨어진다.



## 아이템 67. 최적화는 신중히 하라

> To summarize, do not strive to write fast programs—strive to write good ones; speed will follow. But do think about performance while you’re designing systems, especially while you’re designing APIs, wire-level protocols, and persistent data formats. When you’ve finished building the system, measure its performance. If it’s fast enough, you’re done. If not, locate the source of the problem with the aid of a profiler and go to work optimizing the relevant parts of the system. The first step is to examine your choice of algorithms: no amount of low-level optimization can make up for a poor choice of algorithm. Repeat this process as necessary, measuring the performance after every change, until you’re satisfied.

> 빠른 프로그램을 작성하려 안달하지 말자. 좋은 프로그램을 작성하다 보면 성능은 따라오게 마련이다. 하지만 시스템을 설계할 때, 특히 API, 네트워크 프로토콜, 영구 저장용 데이터 포맷을 설계할 때는 성능을 염두에 두어야 한다. 시스템 구현을 완료했다면 이제 성능을 측정해보라. 충분히 빠르면 그것으로 끝이다. 그렇지 않다면 프로파일러를 사용해 문제의 원인이 되는 지점을 찾아 최적화를 수행하라. 가장 먼저 어떤 알고리즘을 사용했는지를 살펴보자. 알고리즘을 잘못 골랐다면 다른 저수준 최적화는 아무리 해봐야 소용이 없다. 만족할 때까지 이 과정을 반복하고, 모든 변경 후에는 성능을 측정하라.



- 최적화는 좋은 결과보다는 해로운 결과로 이어지기 쉽고, 섣불리 진행하면 특히 더 그렇다.
  - 빠르지도 않고 제대로 동작하지도 않으면서 수정하기는 어려운 소프트웨어를 탄생시키는 것이다.



- 성능 때문에 견고한 구조를 희생하지 말자.
  - **빠른 프로그램보다는 좋은 프로그램**을 작성하라.
  - 좋은 프로그램이지만 원하는 성능이 나오지 않는다면 그 아키텍처 자체가 최적화할 수 있는 길을 안내해줄 것이다.
    - 정보 은닉 원칙을 따르는 좋은 프로그램은 시스템의 나머지에 영향을 주지 않고도 각 요소를 다시 설계할 수 있다(아이템 15).
- 프로그램을 완성할 때까지 성능 문제를 무시하라는 뜻이 아니다. 
  - 구현상의 문제는 나중에 최적화해 해결할 수 있지만, 아키텍처의 결함이 성능을 제한하는 상황이라면 시스템 전체를 다시 작성하지 않고는 해결하기 불가능할 수 있다.
    - 설계 단계에서 성능을 반드시 염두에 두어야 한다.



- 성능을 제한하는 설계를 피하라.
  - 완성 후 변경하기가 가장 어려운 설계 요소는 바로 컴포넌트끼리, 혹은 외부 시스템과의 소통 방식이다. 
    - API, 네트워크 프로토콜, 영구 저장용 데이터 포맷 등이 대표적이다. 
    - 이런 설계 요소들은 완성후에는 변경하기 어렵거나 불가능할 수 있으며, 동시에 시스템 성능을 심각하게 제한할 수 있다.



- API를 설계할 때 성능에 주는 영향을 고려하라.
  - `public` 타입을 가변으로 만들면, 즉 내부 데이터를 변경할 수 있게 만들면 불필요한 방어적 복사를 수없이 유발할 수 있다(아이템 50). 
  - 컴포지션으로 해결할 수 있음에도 상속 방식으로 설계한 `public` 클래스는 상위 클래스에 영원히 종속되며 그 성능 제약까지도 물려받게 된다(아이템 18). 
  - 인터페이스도 있는데 굳이 구현 탑입을 사용하는 것 역시 좋지 않다.
    - 특정 구현체에 종속되게 하여, 나중에 더 빠른 구현체가 나오더라도 이용하지 못하게 된다(아이템 64).



- 다행히 잘 설계된 API는 성능도 좋은 게 보통이다.
  - 그러니 성능을 위해 API를 왜곡하는 건 매우 안 좋은 생각이다.
  - API를 왜곡하도록 만든 그 성능 문제는 해당 플랫폼이나 아랫단 소프트웨어의 다음 버전에서 사라질 수도 있지만, 왜곡된 API와 이를 지원하는 데 따르는 고통은 영원히 계속될 것이다.



## 아이템 68. 일반적으로 통용되는 명명 규칙을 따르라

> To summarize, internalize the standard naming conventions and learn to use them as second nature. The typographical conventions are straightforward and largely unambiguous; the grammatical conventions are more complex and looser. To quote from *The Java Language Specification* [JLS, 6.1], “These conventions should not be followed slavishly if long-held conventional usage dictates other- wise.” Use common sense.

> 표준 명명 규칙을 체화하여 자연스럽게 베어 나오도록 하자. 철자 규칙은 직관적이라 모호한 부분이 적은 데 반해, 문법 규칙은 더 복잡하고 느슨하다. 자바 언어 명세[JLS, 6.1]의 말을 인용하자면 "오랫동안 따라온 규칙과 충돌한다면 그 규칙을 맹종해서는 안 된다." 상식이 이끄는 대로 따르자.



- 자바 명명 규칙은 크게 철자와 문법 두 범주로 나뉜다.



### 철자 규칙

- 철자 규칙은 패키지, 클래스, 인터페이스, 메서드, 필드, 타입 변수의 이름을 다룬다.



- 클래스와 인터페이스(열거 타입과 애너테이션을 포함해)
  - 여러 단어의 첫 글자만 딴 약자나 max, min 처럼 널리 통용되는 줄임말을 제외하고는 단어의 첫 글자만 딴 약자나 max, min처럼 널리 통용되는 줄임말을 제외하고는 단어를 줄여 쓰지 않도록 한다.
  - 약자의 경우 첫 글자만 대문자로 할지 전체를 대문자로 할지는 살짝 논란이 있다.
    - 약자의 경우 첫 글자만 대문자로 할지 전체를 대문자로 할지는 살짝 논란이 있다.
    - 전체를 대문자로 쓰는 프로그래머도 있지만, 그래도 첫 글자만 대문자로 하는 쪽이 훨씬 많다.
      - HttpUrl, HTTPURL 두개 비교해보자.



- 메서드와 필드 이름은 첫 단어가 약자라면 단어 전체가 소문자여야 한다.



- '상수 필드'는 예외로 상수 필드를 구성하는 단어는 모두 대문자로 쓰며 단어 사이는 밑줄로 구분한다.
  - VALUES, NEGATIVE_INFINITY 등
  - 상수 필드는 값이 불변인 static final 필드를 말한다.
  - 이름에 밑줄을 사용하는 요소로는 상수 필드가 유일하다는 사실도 기억해두자.



- 지역변수는 약어를 써도 좋다.
  - 약어를 써도 그 변수가 사용되는 문맥에서 의미를 쉽게 유추할 수 있기 때문이다.
    - i, denom, houseNum 등



- 타입 매개변수 이름은 보통 한 문자로 표현한다.
  - 임의의 타입엔 T를, 컬렉션 원소의 타입은 E를, 맵의 키와 값에는 K와 V를, 예외에는 X를, 메서드의 반환 타입에는 R을 사용한다.



### 문법 규칙

- 문법 규칙은 철자 규칙과 비교하면 더 유연하고 논란도 많다.

- 꼭 언급해둬야 할 특별한 메서드 이름 몇 가지
  - **객체의 타입을 바꿔서 반환**하는 인스턴스 메서드의 이름은 보통 **toType** 형태로 짓는다(`toString`, `toArray` 등).
  - **객체의 내용을 다른 뷰로 보여주는** 메서드(아이템 6)의 이름은 **asType** 형태로 짓는다(`asList` 등).
  - **객체의 값을 기본 타입 값으로 반환**하는 메서드의 이름은 보통 **typeValue** 형태로 짓는다(`intValue` 등).
  - **정적 팩터리**의 이름은 다양하지만 `from`, `of`, `valueOf`, `instance`, `getInstance`, `newInstance`, `getType`, `newType`(아이템 1)을 흔히 사용한다.

