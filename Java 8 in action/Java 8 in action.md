
## Part 1. 기초



### Chapter 02. 동작 파라미터화 코드 전달하기

- 메서드의 파라미터를 늘려가면서 추상화, 람다 등을 적용하는 부분
- 동작 파라미터화
  - 추상화하기
  - 한 개의 파라미터, 다양한 동작
  - 전략 디자인 패턴

- 복잡한 과정 간소화
  - 익명 클래스
  - 람다 표현식 사용



### Chapter 03. 람다 표현식

- 함수형 인터페이스

  - 많은 디폴트 메서드가 있더라도 **추상 메서드가 오직 하나면** 함수형 인터페이스다.
  - 자바 API를 보다가 파라미터에 함수형 인터페이스가 있다?
    - 그러면 사용하는 측에서 람다 표현식을 사용 할 수 있다!
    - 해당 메서드의 시그니처와 일치하는 람다를 전달할 수 있다.

- 특별한 void 호환 규칙

  - 람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다.

  ```java
  // Predicate는 불린 반환값을 갖는다.
  Predicate<String> p = s -> list.add(s);
  // Consumer는 void 반환값을 갖는다.
  Consumer<String> b = s -> list.add(s);
  ```

- 형식 추론

  - `(a1, a2) -> a1.getA().compareTo(a2.getB());` 
    - Apple a1 에서 Apple을 생략
  - 어떤 방법이 좋은지 정해진 규칙은 없다.
    - 가독성이 향상을 기준으로 결정하자.

- 지역 변수의 제약

  - 람다에서 참고하는 지역 변수는 final로 선언되거나 실질적으로 final 처럼 취급되어야 한다.

  - 인스턴스 변수는 힙에, 지역 변수는 스택에 위치

  - 람다에서 여러 스레드를ja 사용한다고 했을 때 스택에 있으면 하나의 스레드가 끝나면 스레드와 함께 사라진다.

  - 자바 수현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다.

    따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴 것이다.

- 인자가 세 개인 생성자의 레퍼런스를 사용하려면 어떻게 해야 할까?

  - 현재 이런 시그너처를 같는 함수형 인터페이스는 제공되지 않으므로 우리가 직접 다은과 같은 함수형 인터페이스를 만들어야 한다.

  ```java
  public interface TriFunction<T, U, V, R> {
    R apply(T t, U u, V v);
  }
  ```

- 람다 표현식을 조합할 수 있는 유용한 메서드

  - 디폴트 메서드

    - 인터페이스의 static 메서드는 디폴트 메서드와 유사하지만 

      인터페이스를 구현하는 클래스에서 메서드를 재정의 할 수 없다는 점이 다릅니다.

    - `Comparator` 에 `reversed()`,`thenComparing()` 등 



## Part 2. 함수형 데이터 처리



### Chapter 04. 스트림 소개

- 자바 8 스트림 API 특징
  - 선언형
    - 더 간결하고 가독성이 좋아진다.
  - 조립할 수 있음
    - 유연성이 좋아진다.
  - 병렬화
    - 성능이 좋아진다.

- 스트림이란?
  - 데이터 처리 연산을 지원 하도록 소스에서 추출된 연속된 요소
  - 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다.
    - 딱 한 번만 탐색할 수 있다.

- 스트림과 컬렉션
  - 컬렉션의 주제는 데이터고 스트림의 주제는 계산
  - DVD처럼 컬렉션은 현재 자료구조에 포함된 모든 값을 계산한 다음에 컬렉션에 추가할 수 있다.
  - 자바 8의 스트림은 스트리밍 비디오처럼 필요할 때 값을 계산한다.



### Chapter 05. 스트림 활용 

- 필터링과 슬라이싱

  - 프레디케이트로 필터링
    - `.filter(Dish::isVegetarian)`
  - 고유 요소 필터링
    - `.distinct()`
  - 스트림 축소
    - `.limit(3)`
  - 요소 건너뛰기
    - `.skip(2)`

- 매핑

  - flatMap 메서드는 스트림의 각 값을 다른 스트림으로 만든 다음에

    모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다.

  - 프로그래머스 - 완전탐색 - 숫자 야구

    - flatMap() 안쓰면 String[]이 반환 된다.

    ```java
    List<String> list = Arrays.stream(baseball)
      												.map(i -> String.valueOf(i[0]).split(""))
      												.flatMap(Arrays::stream)
      												.distinct()
      												.collect(Collectors.toList());
    ```

    ```java
    List<int[]> pairs = 
      numbers1.stream()
              .flatMap((Integer i) -> numbers2.stream()
                       												.map((Integer j) -> new int[]{i, j}))
              .filter(pair -> (pair[0] + pair[1]) % 3 == 0)
              .collect(toList());
    ```

- 검색과 매칭

  - 프레디케이트가 적어도 한 요소와 일치하는지 확인
    - `.anyMatch()`
  - 프레디케이트가 모든 요소와 일치하는지 검사
    - `.allMatch()`
    - `.noneMatch()`
  - 임의의 요소를 반환
    - `.findAny()`
      - 병렬 실행시에 순서가 상관 없으면 이걸 쓴다네
      - Optional
        - `isPresent()`
        - `ifPresent(Consumer<T> block)`
        - `T get()`
        - `T orElse(T other)`
  - 첫 번째 요소 찾기
    - `.findFirst()`

- 리듀싱

  - 모든 스트림 요소를 처리해서 값으로 도출하는걸 **리듀싱 연산**이라고 한다.
  - 함수형 프로그래밍 언어 용어로는 이 과정이 마치 종이(우리 스트림)를 작은 조각이 될 때까지 반복해서 접는 것과 비슷하다는 의미로 **폴드**라고 부른다.
  - 요소의 합
    - reduce는 두 개의 인수를 갖는다.
      - 초깃값
      - 두 요소를 조합해서 새로은 값을 만드는 `BinaryOperator<T>`
      - `numbers.stream.reduce(1, (a, b) -> a * b);`
      - `numbers.stream.reduce(1, Integer::sum;`
    - 초깃값 없음
      - 스트림에 아무 요소도 없는 상황이 있을 수 있으니 결과는 Optional 객체로 감싸서 반환된다.
      - `Optional<Integer> sum = numbers.stream.reduce((a, b) -> a * b);`
  - 최댓값과 최소값
    - `Optional<Integer> max = numbers.stream.reduce(Integer::max;`
    - `Optional<Integer> min = numbers.stream.reduce(Integer::min;`

- 스트림 연산: 상태 없음과 상태 있음

  - 내부 상태를 갖지 않는 연산

    - map, filter

  - 내부 상태의 크기가 한정되어 있는 연산

    - reduce, sum, max

  - 내부 상태를 갖는 연산

    - sorted, distinct

    - 데이터 스트림의 크기가 크거나 무한이라면 문제가 생길 수 있다.

      - 예를 들어 모든 소수를 포함하는 스트림을 역순으로 만들면 어떤 일이 일어날까?

        첫 번째 요소로 가장 큰 소수, 즉 세상에 존재하지 않는 수를 반환해야 한다.

- 실전 연습

  - 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오.

    - `reduce()` 를 활용해서 각각의 이름을 하나의 문자열로 연결하여 결국 모든 이름 연결

    ```java
    String traderStr = 
      transactions.stream()
                  .map(transaction -> transaction.getTrader().getName())
                  .distinct()
                  .sorted()
                  .reduce("", (n1, n2) -> n1 + n2);
    
    String traderStr = 
      transactions.stream()
                  .map(transaction -> transaction.getTrader().getName())
                  .distinct()
                  .sorted()
                  .collect(joining());
    ```

  - 전체 트랜잭션 중 최솟값은 얼마인가?

    - 역시 `reduce()` 를 사용하는 부분 체크

    ```java
    Optional<Transaction> smallestTransaction = 
      transactions.stream()
      						.reduce((t1, t2) -> t1.getValue() < t2.getValue() ? t1 : t2);
    
    Optional<Transaction> smallestTransaction =
      transactions.stream()
      						.min(comparing(Transaction::getValue));
    ```

- 숫자형 스트림

  - 기본형 특화 스트림

    - 숫자 스트림으로 매핑
      - `mapToInt()`, `mapToDouble`, `mapToLong` 세자지를 주로 많이 쓴다고 한다.
    - 객체 스트림으로 복원하기
      - `boxed()`

  - 숫자 범위

    - `range()`
      - 시작값과 종료값이 결과에 포함되지 않는다.
    - `rangeClosed`
      - 시작값과 종료값이 결과에 포함된다.

  - 숫자 스트림 활용: 피타고라스 수

    ```java
    Stream<int[]> pythagoreanTriples =
      IntStream.rangeClosed(1, 100)
               .boxed()
               .flatMap(a -> IntStream
                        		  .rangeClosed(a, 100)
                              .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
                              .boxed()
                              .map(b -> new int[]{a, b, (int) Math.sqrt(a * a + b * b)})
               );
    ```

    

### Chapter 06. 스트림으로 데이터 수집 

- 인트로

  - 중간 연산은 스트림 파이프라인을 구성하며, 스트림의 요소를 소비하지 않는다.

  - 최종 연산은 스트림 파이프라인을 최적화하면서 계산 과정을 짧게 생략하기도 한다.

  - 이 장에서는 reduce가 그랬던 것처럼 collect 역시 다양한 요소 누적 방식을 

    인수로 받아서 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행할 수 있을을 설명한다.

- 리듀싱과 요약

  - `menu.stream().collect(counting());`

  - 스트림값에서 최댓값과 최솟값 검색

    - `menu.stream().collect(maxBy(Comparator.comparingInt(Dish::getCalories)));`

  - 요약 연산

    - `menu.stream().collect(summingInt(Dish::getCalories));`
    - `menu.stream().collect(averagingInt(Dish::getCalories));`
    - `menu.stream().collect(summarizingInt(Dish::getCalories));`

  - 문자열 연산

    - ` menu.stream().map(Dish::getName).collect(joining());`

    - ` menu.stream().collect(joining());`

      - joining 메서드는 내부적으로 StringBuilder를 이용해서 문자열을 하나로 만든다.

      - Dish 클래스가 요리명을 반환하는 toString 메서드를 포함하고 있다면 

        다음 코드에서 보여주는 것처럼 map으로 각 요리의 이름을 추출하는 과정을 생략할 수 있다.

    - `menu.stream().map(Dish::getName).collect(joining(", "));`

  - 범용 리듀싱 요약 연산

    - 컬렉션 프레임워크 유연성: 같은 연산도 다양한 방식으로 수행할 수 있다.
      - 초깃값, 변환 함수, 합계 함수
      - `menu.stream().collect(reducing(0, Dish::getCalories, (Integer i, Integer j) -> i + j));`
      - `menu.stream().collect(reducing(0, Dish::getCalories, Integer::sum));`

- 그룹화

  ```java
  menu.stream().collect(groupingBy(Dish::getType));
  // Dishes grouped by type: {OTHER=[french fries, rice, season fruit, pizza], MEAT=[pork, beef, chicken], FISH=[prawns, salmon]}
  
  menu.stream().collect(
    groupingBy(dish -> {
      if (dish.getCalories() <= 400) return CaloricLevel.DIET;
      else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
      else return CaloricLevel.FAT;
    })
  );
  // Dishes grouped by caloric level: {DIET=[chicken, rice, season fruit, prawns], NORMAL=[beef, french fries, pizza, salmon], FAT=[pork]}
  ```

  - 다수준 그룹화

    ```java
    menu.stream().collect(
      groupingBy(Dish::getType,
                 groupingBy((Dish dish) -> {
                   if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                   else if (dish.getCalories() <= 700) 
                     return CaloricLevel.NORMAL;
                   else return CaloricLevel.FAT;
                 } )
                )
    );
    // Dishes grouped by type and caloric level: {OTHER={DIET=[rice, season fruit], NORMAL=[french fries, pizza]}, MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]}, FISH={DIET=[prawns], NORMAL=[salmon]}}
    ```

  - 서브그룹으로 데이터 수집

    ```java
    menu.stream().collect(groupingBy(Dish::getType, counting()));
    // Count dishes in groups: {OTHER=4, MEAT=3, FISH=2}
    
    menu.stream().collect(
      groupingBy(Dish::getType, maxBy(comparingInt(Dish::getCalories)));
    // Most caloric dishes by type: {OTHER=Optional[pizza], MEAT=Optional[pork], FISH=Optional[salmon]}
    // Optional 객체로 감싸서 반환
    ```

    - 컬렉터 결과를 다른 형식에 적용하기

      ```java
      menu.stream().collect(
        groupingBy(Dish::isVegetarian,
                       collectingAndThen(
                         maxBy(comparingInt(Dish::getCalories)),
                         Optional::get)));
      ```

    - groupingBy와 함께 사용하는 다른 컬렉터 예제

      - 그룹핑을 하고 하위에 또 그룹핑(`groupingBy`)을 할 수도 있고 컬렉션으로 묶어서 반환 할 수도 있고(`mapping()`) 

      ```java
      menu.stream().collect(groupingBy(Dish::getType,
                      summingInt(Dish::getCalories)));
      // Map<Dish.Type, Integer>
      
      menu.stream().collect(
                      groupingBy(Dish::getType, mapping(
                              dish -> { 
                                if (dish.getCalories() <= 400) 
                                  return CaloricLevel.DIET;
                             	 else if (dish.getCalories() <= 700) 
                                	return CaloricLevel.NORMAL;
                             	 else 
                                 	return CaloricLevel.FAT; },
                              toSet() )));
      // Map<Dish.Type, Set<CaloricLevel>>
      // toCollection(HashSet::new)
      ```

- 분할

  ```java
  menu.stream().collect(partitioningBy(Dish::isVegetarian));
  // Dishes partitioned by vegetarian: {false=[pork, beef, chicken, prawns, salmon], true=[french fries, rice, season fruit, pizza]}
  
  menu.stream().collect(partitioningBy(Dish::isVegetarian, groupingBy(Dish::getType)));
  // Vegetarian Dishes by type: {false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]}, true={OTHER=[french fries, rice, season fruit, pizza]}}
  
  menu.stream().collect(
                  partitioningBy(Dish::isVegetarian,
                          collectingAndThen(
                                  maxBy(comparingInt(Dish::getCalories)),
                                  Optional::get)));
  // Most caloric dishes by vegetarian: {false=pork, true=pizza}
  ```

  - Collectors 클래스의 정적 팩토리 메서드
    - 전부 Collector 인터페이스를 리턴해서 인자값이 Collector 인터페이스인 메소드에서 유용하게 쓸 수 있다.
      - `public static Collector groupingBy(Function classifier, Collector downstream)`
        - Collector 리턴, 인자값에도 Collector
    - 졸라게 많다. API 참고. 



















## Part 3. 효과적인 자바 8 프로그래밍



## Part 4. 자바 8의 한계를 넘어서

