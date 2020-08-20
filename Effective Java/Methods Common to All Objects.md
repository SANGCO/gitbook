

## 인트로

- `Object`에서 `final`이 아닌 메서드(`equals`, `hashCode`, `toString`, `clone`, `finalize`)는 모두 재정의(`overriding`)를 염두에 두고 설계된 것이다.
  - 그래서 `Object`를 상속하는 클래스, 즉 모든 클래스는 이 메서드들을 일반 규약에 맞게 재정의해야 한다.
- 메서드를 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스들(`HashMap`, `HashSet`등)이 오작동하게 만들 수 있다.
- 이번 장에서는 `final`이 아닌 `Object` 메서드들을 언제 어떻게 재정의해야 하는지를 다룬다.
  - `finalize`는 앞에서 봤으니깐 언급 안한다.
  - `Comparable.compareTo`의 경우 `Object`의 메서드는 아니지만 성격이 비슷하여 이번 장에서 함께 다룬다.



## 아이템 10. equals는 일반 규약을 지켜 재정의하라

> In summary, don’t override the equals method unless you have to: in many cases, the implementation inherited from Object does exactly what you want. If you do override equals, make sure to compare all of the class’s significant fields and to compare them in a manner that preserves all five provisions of the equals contract.


> 꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다. 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.



### equals를 재정의하지 않는 것이 최선인 상황

- 그냥 두면 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.
  - `Object`에 재정의 되어있는 `equals` 메소드는 `==` 연산자로 주소를 비교 하기 때문에

```java
public boolean equals(Object obj) {
  return (this == obj);
}
```



- 재정의하지 않는 것이 최선인 상황

  - 각 인스턴스가 **본질적으로 고유**하다.

    - 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당한다.
    - `Thread`가 좋은 예

  - 인스턴스의 '**논리적 동치성**(logical equality)'을 검사할 일이 없는 경우

    - `java.util.regex.Pattern`은 `equals`를 재정의해서 두 `Pattern`의 인스턴스가 같은 정규표현식을 나타내는지를 검사 할 수 있지만 논리적 동치성 검사가 필요 없다고 생각하면 `Object`의 기본 `equals`를 사용할 수도 있다.

  - 상위 클래스에서 재정의한 `equals`가 하위 클래스에도 딱 들어맞는다면

    - 대부분의 `Set` 구현체는 `AbstractSet`이 구현한 `equals`를 상속받아 쓴다.
- `List` 구현체들은 `AbstractList`로부터, `Map` 구현체들은 `AbstractMap`으로부터

- 클래스가 `private`이거나 `package-private`이고 `equals` 메서드를 호출할 일이 없다면

```java
// 위험을 철저히 회피하는 스타일이라 equals가 실수로라도 호출되는 걸 막고 싶다면
@Override public boolean equals(Object o) {
	throw new AssertionError(); //호출 금지!
}
```



### equals를 재정의해야 할 때는 언제일까?

- **객체 식별성**(object identity; 두 객체가 물리적으로 같은가)이 아니라 **논리적 동치성**을 확인해야 하는데, 상위 클래스의 `equals`가 논리적 동치성을 비교하도록 재정의되지 않았을 때
- 두 **값 객체**를 `equals`로 비교하는 프로그래머는 객체가 같은지가 아니라 값이 같은지를 알고 싶어 할 것이다.

  - `equals`가 논리적 동치성을 확인하도록 재정의해두면, 그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부응함은 물론 `Map`의 키와 `Set`의 원소로 사용할 수 있게 된다.
- 값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 **인스턴스 통제 클래스**(아이템 1)라면 `equals`를 재정의하지 않아도 된다.
  - `Enum`(아이템 34)



### equals 메서드를 재정의할 때 반드시 따라야하는 일반 규약

- `equals` 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.
  - **반사성**(reflexivity)
    - null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
  - **대칭성**(symmetry)
    - null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
  - **추이성**(transitivity)
    - null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
  - **일관성**(consistency)
    - null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
  - **null-아님**
    - null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.



- 반사성(reflexivity)

  - 반사성은 단순히 말하면 객체는 자기 자신과 같아야 한다는 뜻.
    - 일부러 어기는 경우가 아니라면 만족시키지 못하는게 더 어려워 보이네.



- 대칭성(symmetry)

  - 대칭성은 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻.

```java
// 대칭성 위배!
@Override public boolean equals(Object o) {
  if (o instanceof CaseInsensitiveString)
    return s.equalsIgnoreCase(
    ((CaseInsensitiveString) o).s);
  if (o instanceof String)  
    // 한 방향으로만 작동한다!
    // String의 equals에 CaseInsensitiveString을 던지면 당연히 false가 나오겠지?
    return s.equalsIgnoreCase((String) o);
  return false;
}
```

```java
// 수정한 equals 메서드
// CaseInsensitiveString의 equals를 String과도 연동하겠다는 허황된 꿈을 버리자.
@Override public boolean equals(Object o) {
  return o instanceof CaseInsensitiveString &&
    ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```



- 추이성(transitivity) 

  - 추이성은 첫 번째 객체와 두 번재 객체가 같고, 두 번째 개체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻이다.

```java
// 코드 10-3 잘못된 코드 - 추이성 위배!
// 대칭성은 지켜지지만 추이성에서 깨진다.
@Override public boolean equals(Object o) {
  if (!(o instanceof Point))
    return false;

  // o가 일반 Point면 색상을 무시하고 비교한다.
  if (!(o instanceof ColorPoint))
    return o.equals(this);

  // o가 ColorPoint면 색상까지 비교한다.
  return super.equals(o) && ((ColorPoint) o).color == color;
}
```

- 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.

- 리스코프 치환 원칙(Liskov substitution principle)
  - 어떤 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.

```java
// 잘못된 코드 - 리스코프 치환 원칙 위배!
// getClass를 씀으로서 Point의 하위타입이 들어오면 Point의 x, y의 값과 상관없이 무조건 false를 반환 
@Override public boolean equals(Object o) {
	if (o == null || o.getClass() != getClass())
    return false;
  Point p = (Point) o;
  return p.x == x && p.y == y;
}
```

- 구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법이 하나 있다.
  - "상속 대신 컴포지션을 사용하라"는 아이템 18의 조언을 따르면 된다.

```java
// 코드 10-5 equals 규약을 지키면서 값 추가하기
public class ColorPoint {
		/**
     * Point를 상속하는 대신 ColorPoint의 private 필드로	
     */
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
     * 이 ColorPoint의 Point 뷰를 반환한다.
     */
    public Point asPoint() {
        return point;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }

    @Override public int hashCode() {
        return 31 * point.hashCode() + color.hashCode();
    }
}
```

- 자바 라이브러리에도 구체 클래스를 확장하고 값을 추가한 클래스가 종종 있다.
  - `java.sql.Timestamp`는 `java.util.Date`를 확장한 후 `nanoseconds` 필드를 추가했다.
    - `Timestamp`의 `equals`는 대칭성 위배
    - `Date` 객체와 `Timestamp` 객체를 한 컬렉션에 넣거나 서로 섰어 사용하면 **엉뚱하게 동작**할 수 있다.
    - `Timestamp`를 이렇게 설계한 것은 실수니 **절대 따라 해서는 안 된다**.

- 추상 클래스의 하위 클래스에서라면 `equals` 규약을 지키면서도 값을 추가할 수 있다.
  - 상위 클래스를 직접 인스턴스로 만드는 게 불가능하다면 지금까지 이야기한 문제들은 일어나지 않는다.
  - 그렇다고 `Rectangle`과 `Circle`을 추상 클래스인 `Shape` 타입으로 한 컬렉센에 넣으면 당연히 안되겠지?



- 일관성(consistency)

  - 일관성은 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻
  - 클래스를 작성할 때는 불변 클래스로 만드는 게 나을지를 심사숙고하자(아이템 17).
    - 불변 클래스로 만들기로 했다면 `equals`가 한번 같다고 한 객체와는 영원히 같다고 답하고, 다르다고 한 객체와는 영원히 다르다고 답하도록 만들어야 한다.
  - 클래스가 불변이든 가변이든 `equals`의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.
    - `java.net.URL`의 `equals`는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교한다.
      - 호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 보장할 수 없다.
      - URL의 `equals`를 이렇게 구현한것은 커다란 실수였으니 **절대 따라 해서는 안 된다**.



- null-아님

  - null-아님은 이름처럼 모든 객체가 `null`과 같지 않아야 한다는 뜻

```java
// 명시적 null 검사 - 필요 없다!
@Override public boolean equals(Object o) {
    if (o == null) 
      	return false;
  
    ...
      
}

// 묵시적 null 검사 - 이쪽이 낫다!
@Override public boolean equals(Object o) {
    if (!(o instanceof MyType))  
      	return false;
    MyType mt = (MyType) o;
  
    ...
      
}
```



### 양질의 equals 메서드 구현 방법

- 지금까지의 내용을 종합해서 양질의 `equals` 메서드 구현 방법을 단계별로 정리해자.
  - **1단계** `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    - 자기자신이면 `true` 반환
  - **2단계** `instanceof` 연산자로 입력이 올바른 타입인지 확인한다.
    - 그렇지 않다면 `false`를 반환
    - 이때 올바른 타입은 `equals`가 정의된 클래스인 것이 보통
    - 가끔은 그 클래스가 구현한 특정 인터페이스가 될 수도 있다.
      - ex) `Set`, `List`, `Map`, `Map.Entry` 등의 컬렉션 인터페이스들이 여기 해당
  - **3단계** 입력을 올바른 타입으로 형변환한다.
    - 2단계에서 `instanceof` 검사를 했기 때문에 이 단계는 100% 성공
  - **4단계** 입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사한다.
    - 모든 필드가 일치하면 `true`를, 하나라도 다르면 `false`를 반환
    - 2단계에서 인터페이스를 사용했다면 입력의 필드 값을 가져올 때도 그 인터페이스의 메서드를 사용해야 한다.

```java
// 코드 10-6 전형적인 equals 메서드의 예
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix   = rangeCheck(prefix,   999, "프리픽스");
        this.lineNum  = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override public boolean equals(Object o) {
        // 1단계
      	if (o == this)
            return true;
        // 2단계
      	if (!(o instanceof PhoneNumber))
            return false;
        // 3단계
      	PhoneNumber pn = (PhoneNumber)o;
        // 4단계
      	return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    // 나머지 코드는 생략 - hashCode 메서드는 꼭 필요하다(아이템 11)!
}
```



- `float`와 `double`을 제외한 **기본 타입 필드는 `==` 연산자로** 비교하자.
- **참조 타입 필드는 각각의 `equals` 메서드로** 비교하자.
- **`float`와 `double` 필드는 각각 정적 메서드**인 `Float.compare(float, float)`와 `Double.compare(double, double)`로 비교하자.
  - `Float.NaN`, `-0.0f`, 특수한 부동소수 값 등을 다뤄야 하기 때문에 특별 취급
  - `Float.equals`와 `Double.equals` 메서드를 대신 사용할 수도 있지만, 이 메서드들은 **오토박싱**을 수반할 수 있으니 **성능**상 좋지 않다.
- **어떤 필드를 먼저 비교하느냐가 `equals`의 성능을 좌우하기도 한다.**
  - 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 싼 (혹은 둘 다 해당하는) 필드를 먼저 비교하자.
  - 동기화용 락(lock) 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면 안 된다.
  - 핵심 필드로부터 계산해낼 수 있는 파생 필드 역시 굳이 비교할 필요는 없지만, 파생 필드를 비교하는 쪽이 더 빠를 때도 있다.

- `equals`를 다 구현했다면 세 가지만 자문해보자. 대칭적인가? 추이성이 있는가? 일관적인가?
  - 자문에서 끝내지 말고 단위 테스트를 작성해 돌려보자.



### 주의사항

- `equals`를 재정의할 땐 `hashCode`도 반드시 재정의하자(아이템 11).
- 너무 복잡하게 해결하려 들지 말자.
  - 필드들의 동치성만 검사해도 `equals`규약을 어렵지 않게 지킬 수 있다.
  - 오히려 너무 공격적으로 파고들다가 문제를 일으키기도 한다.
    - 예컨대 `File` 클래스라면, 심볼릭 링크를 비교해 같은 파일을 가리키는지를 확인하려 들면 안 된다.
- `Object` 외의 타입을 매개변수로 받는 `equals` 메서드는 선언하지 말자.
  - 입력 타입이 `Object`가 아니므로 재정의가 아니라 다중정의(아이템 52)한 것이다.
  - '타입을 구체적으로 명시한' `equals`는 오히려 해가 될 수 있다.
  - 이번 절 예제 코드들에서처럼 `@Override` 애너테이션을 일관되게 사용하면 이러한 실수를 예방할 수 있다. 

```java
// 잘못된 예 - 입력 타입은 반드시 Object여야 한다!
// public boolean equals(Object o) { 
public boolean equals(MyClass o) {
  ...
}

// 여전히 잘못된 예 - 컴파일되지 않음
@Override public boolean equals(MyClass o) {
  ...
}
```



- 사람이 직접 작성하는 것보다는 IDE에 맡기는 편이 낫다.
  - 적어도 사람처럼 부주의한 실수를 저지르지는 않으니깐.
  - 저자는 AutoValue 프레임워크를 추천한다.
    - IDE도 같은 기능을 제공하지만 AutoValue만큼 깔끔하거나 읽기 좋지는 않다고
    - 또한 IDE는 나중에 클래스가 수정된 걸 자동으로 알아채지는 못하니 테스트 코드를 작성해둬야 한다.
    - 인텔리제이 자체 기능 체크, 인텔리제이 AutoValue plugin 체크



## 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라

> In summary, you *must* override hashCode every time you override equals, or your program will not run correctly. Your hashCode method must obey the general contract specified in Object and must do a reasonable job assigning unequal hash codes to unequal instances. This is easy to achieve, if slightly tedious, using the recipe on page 51. As mentioned in Item 10, the AutoValue framework provides a fine alternative to writing equals and hashCode methods manually, and IDEs also provide some of this functionality.


> equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다. 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다. 이렇게 구현하기가 어렵지는 않지만 조금 따분한 일이긴 하다(68쪽의 요령을 참고하자). 하지만 걱정마시라. 아이템 10에서 이야기한 AutoValue 프레임워크를 사용하면 멋진 equals와 hashCode를 자동으로 만들어준다. IDE들도 이런 기능을 일부 제공한다.



- `equals`를 재정의한 클래스 모두에서 `hashCode`도 재정의해야 한다.
- 그렇지 않으면 `HashCode` 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 `HashMap`이나 `HashSet` 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.
- `Object` 명세에서 발췌한 규약
- `equals` 비교에 사용되는 정보가 변경되지 않았다면, **애플리케이션이 실행되는 동안** 그 객체의 hashCode 메서드는 몇 번을 호출해도 **일관되게 항상 같은 값을 반환**해야 한다. 
  - 단 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
  - `equals(Object)`가 두 객체를 **같다고 판단**했다면 두 객체의 `hashCode`는 똑같은 값을 반환해야 한다.
  - `equals(Object)`가 두 객체가 **다르다고 판단**했더라도, 두 객체의 `hashCode`가 서로 다른 값을 반환할 필요는 없다. 
  - 단, 다른 객체에 대해서는 다른 값을 반환해야 **해시테이블의 성능**이 좋아진다.
- `hashCode` 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다. 
  - **논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**
  - **equals는 물리적으로 다른 두 객체를 논리적으로는 같다고 할 수 있다.**
  - 하지만 Object의 기본 hashCode 메서드는 이 둘이 전혀 다르다고 판단하여, 규약과 달리 (무작위처럼 보이는) 서로 다른 값을 반환한다.

```java
public static void main(String[] args) {
    Map<PhoneNumber, String> m = new HashMap<>();
    m.put(new PhoneNumber(707, 867, 5309), "제니");
    System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
}
// hashCode를 재정의 안하면 "제니"가 안나온다.
// HashMap은 해시코드가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화 되어 있다.
```



- 이상적인 해시 함수는 주어진 (서로 다른) 인스턴스들을 32비트 정수 범위에 균일하게 분배

- 좋은 `hashCode`를 작성하는 간단한 요령

  - **1단계** `int` 변수 result를 선언한 후 값 c로 초기화한다.

  - **2단계** 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.

    a. 해당 필드의 해시코드 c를 계산한다.

    b. 2단계 a에서 계산한 해시코드 c로 result를 갱신한다. `result = 31 * result + c;`

  - **3단계** result를 반환한다.

```java
// 코드 11-2 전형적인 hashCode 메서드 (70쪽)
// 필드의 타입이 다 short다.
@Override public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```



- 파생 필드는 해시코드 계산에서 제외해도 된다.
  - 즉, 다른 필드로부터 계산해 낼 수 있는 필드는 모두 무시해도 된다.
- `equals` 비교에 사용되지 않은 필드는 '반드시' 제외해야 한다.

- `Objects` 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 `hash`를 제공한다.
  - 아쉽게도 속도는 더 느리다.
    - 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야 하기 때문
  - `Objects.hash()` 메서드는 성능에 민감하지 않은 상황에서만 사용하자.

```java
// 코드 11-3 한 줄짜리 hashCode 메서드 - 성능이 살짝 아쉽다.
@Override public int hashCode() {
  return Objects.hash(lineNum, prefix, areaCode);
}
```



- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 한다.
  - 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 **인스턴스가 만들어질 때 해시코드를 계산**해둬야 한다.
  - 해시의 키로 사용되지 않는 경우라면 `hashCode`가 처음 불릴 때 계산하는 **지연 초기화(lazy initialization) 전략**은 어떨까?
    - 필드를 지연 초기화하려면 그 클래스를 스레드 안전하게 만들도록 신경 써야 한다(아이템 83).

```java
// 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 한다.
// 해시코드가 처음 호출될 때 변수 hashCode에 값을 저장
private int hashCode; // 자동으로 0으로 초기화된다.

@Override public int hashCode() {
  int result = hashCode;
  if (result == 0) {
    result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    hashCode = result;
  }
  return result;
}
```



- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.
  - 속도야 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있다.
  - 특히 어떤 필드는 특정 영역에 몰린 인스턴스들의 해시코드를 넓은 범위로 고르게 퍼트려주는 효과가 있을지도 모른다.
- `hashCode`가 반환하는 값의 생성 규칙을 API사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고 추후에 계산 방식을 바꿀 수도 있다.



## 아이템 12. toString을 항상 재정의하라

> To recap, override Object’s toString implementation in every instantiable class you write, unless a superclass has already done so. It makes classes much more pleasant to use and aids in debugging. The toString method should return a concise, useful description of the object, in an aesthetically pleasing format.


> 모든 구체 클래스에서 Object의 toString을 재정의하자. 상위 클래스에서 이미 알맞게 재정의한 경우는 예외다. toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다. toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.



- `Object`의 기본 `toString` 메서드는 단순히 **클래스_ 이름@16진수로_ 표현한_ 해시코드** 를 반환한다.
- `toString`의 일반 규약에 따르면 '**간결하면서 사람이 읽기 쉬운 형태의 유익한 정보**' 를 반환해야 한다.
- 또한 `toString`의 규약은 "**모든 하위 클래스에서 이 메서드를 재정의하라**"고 한다.
- `equals`와 `hashCode` 규약(아이템 10, 11)만큼 대단히 중요하진 않지만, `toString`을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.
  - `toString` 메서드는 객체를 `println`, `printf`, 문자열 연결 연산자(`+`), `assert` 구문에 넘길 때, 혹은 디버거가 객체를 출력할 때 자동으로 불린다.
    - 내가 직접 호출하지 않더라도 오류 메세지 로깅 등 다른 어딘가에서는 쓰인다는 거지. 
- 실전에서 `toString`은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.
  - 이상적으로는 스스로를 완벽히 설명하는 문자열이어야 한다.

```java
Assertion failure: expected {abc, 123}, but was {abc, 123}.
// 단언 실패: 예상값 {abc, 123}, 실제값 {abc, 123}.
```



- 포맷을 명시하든 아니든 여러분의 의도는 명확히 밝혀야 한다.
  - 포맷을 명시하기로 했다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 **정적 팩터리나 생성자를 함께 제공**해주면 좋다. 
    - 자바 플랫폼의 많은 값 클래스가 따르는 방식이기도 하다. 
      - `BigInteger`, `BigDecimal`과 대부분의 기본 타입 클래스가 여기 해당된다.

```java
public class BigInteger extends Number implements Comparable<BigInteger> {
		
  	...
  
  	public String toString(int radix) {
      	
      	...
          
        if (this.mag.length <= 20) {
          return this.smallToString(radix);
        } else {
          StringBuilder sb = new StringBuilder();
          if (this.signum < 0) {
            // 여기서 static toString으로 넘긴다.
            toString(this.negate(), sb, radix, 0);
            sb.insert(0, '-');
          } else {
            toString(this, sb, radix, 0);
          }

          return sb.toString();
        }  
    }
  
  	private static void toString(BigInteger u, StringBuilder sb, int radix, int digits) { ... }
      
  	public String toString() {
      	// 기본은 10진수, 원하는 형태가 있으면 toString(int radix)에 파라미터로 던지면 된다.
        return this.toString(10);
    }
}
```

```java
public final class PhoneNumber {

  	...

		// 포멧을 명시한 경우
      
		/**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
     * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
     *
     * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
     * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
     * 전화번호의 마지막 네 문자는 "0123"이 된다.
     */
    @Override public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }
  
  
  	// 포멧을 명시하지 않은 경우
      
		/**
     * 이 약물에 관한 대략적인 설명을 반환한다.
     * 다음은 이 설명의 일반적인 형태이나,
     * 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
     * 
     * "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
     */
    @Override public String toString() { ... }
  
}  
  
```



- 포맷 명시 여부와 상관없이 `toString`이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.
  - `PhoneNumber` 클래스는 지역 코드, 프리픽스, 가입자 번호용 접근자를 제공해야 한다.
- 접근자를 제공하지 않으면 (변경될 수 있다고 문서화했더라도) 그 포맷이 사실상 준-표준 API나 다름없어진다.
- 정적 유틸리티 클래스(아이템 4)는 `toString`을 제공할 이유가 없다. 또한, 대부분의 열거 타입(아이템 34)도 자바가 이미 완벽한 `toString`을 제공하니 따로 재정의하지 않아도 된다.
- 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스라면 `toString`을 재정의해줘야 한다.
  - 예컨대 대다수의 컬렉션 구현체는 추상 컬렉션 클래스들의 `toString` 메서드를 상속해 쓴다.
    - `ArrayList` 클래스에는 `toString` 메서드 없네.
      - `abstract class AbstractList<E>` 위에 `abstract class AbstractCollection<E>` 여기서 `toString` 재정의 
- 역시 AutoValue 프레임 위크는 `toString`도 생성해준다(대부분의 IDE도 마찬가지다).
  - 비록 자동 생성에 적합하지는 않더라도 객체의 값에 관해 아무것도 알려주지 않는 `Object`의 `toString` 보다는 자동 생성된 `toString`이 훨씬 유용하다.



## 아이템 13. clone 재정의는 주의해서 진행하라 

> Given all the problems associated with Cloneable, new interfaces should not extend it, and new extendable classes should not implement it. While it’s less harmful for final classes to implement Cloneable, this should be viewed as a per- formance optimization, reserved for the rare cases where it is justified (Item 67). As a rule, copy functionality is best provided by constructors or factories. A nota- ble exception to this rule is arrays, which are best copied with the clone method.


> Cloneable이 몰고 온 모든 문제를 되짚어봤을 때, 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다. final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야 한다(아이템 67). 기본 원칙은 '복제 기능은 생성자와 팩터리를 이용하는 게 최고'라는 것이다. 단, 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.



- `Cloneable`은 복제해도 되는 클래스임을 명시하는 용도의 **믹스인 인터페이스**(mixin interface, 아이템 20)
  - 믹스인(mixin)
    - 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 **특정 선택적 행위를 제공한다고 선언**하는 효과를 준다.
    - 클래스의 주된 기능에 선택적 기능을 '혼합(mixed in)'한다고 해서 미스인이라 부른다.
    - 클래스가 `Comparable`을 구현 한다는건 클래스의 인스턴스 끼리는 순서를 정할 수 있다고 선언하는 것이다.
    - **믹스인 인터페이스**는 타입 체크에 쓰이는 **마커 인터페이스**와는 또 다른 느낌인거 같다.
    - `Serializable`, `Cloneable`은 메서드가 없지만 `Comparable`에는 `compareTo()`가 있다.
  - 아쉽게도 의도한 목적을 제대로 이루지 못했다.
    - `clone()` 메서드는 `Cloneable`이 아닌 `Object`에 선언 되어있다.
      - `protected`로 선언되어 있어서 외부 객체에서 호출 할 수도 없다.
  - 여러 문제점에도 불구하고 `Cloneable` 방식은 널리 쓰이고 있어서 잘 알아두는 것이 좋다. 



### 메서드 하나 없는 Cloneable 인터페이스는 무슨 일을 할까

- `Cloneable` 인터페이스는 `Object`의 `protected` 메서드인 `clone`의 **동작 방식을 결정**한다.
- `Cloneable`을 구현한 클래스의 인스턴스에서 clone을 호출
  - 그 객체의 필드들을 하나하나 복사한 객체를 반환
- `Cloneable`을 구현하지 않은 클래스의 인스턴스에서 `clone`을 호출
  - `CloneNotSupportedException`을 던진다.
- 인터페이스를 구현한다는 것은 일반적으로 인터페이스에서 정의한 기능을 해당 클래스가 제공한다고 선언하는 행위다.
  - 그런데 `Cloneable`의 경우에는 상위 클래스에 정의된 `protected` 메서드의 **동작 방식을 변경**한 것이다.
    - `Object` 클래스에 `clone()` 메서드가 호출은 되는데 `Cloneable` 인터페이스를 구현하고 안하고에 따라 동작이 달라진다는 의미임.
  - 보통 인터페이스를 구현한다는 것은 인터페이스에 선언되어 있는 메서드를 구현하는데 `Cloneable`에 경우는 상위 클래스에 정의된 `protected` 메서드를 오버라이드한다. 독특할세. 
    - 인터페이스를 상당히 이례적으로 사용한 예이니 따라 하지는 말자.

- 실무에서 `Cloneable`을 구현한 클래스는 `clone()` 메서드를 `public`으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다.
  - 상위 클래스의 `clone()` 메서드는 `protected`로 선언되어 있어 `Cloneable`을 구현한 클래스가 `public`으로 `clone()` 메서드를 제공하지 않으면 `clone()` 메서드를 호출할 수도 없다. 



### 가변 상태를 참조하지 않는 클래스용 clone 메서드

```java
public final class PhoneNumber implements Cloneable {
  	private final short areaCode, prefix, lineNum;

		...	
  
		// 코드 13-1 가변 상태를 참조하지 않는 클래스용 clone 메서드
    // 이 메서드가 동작하게 하려면 위와 같이 Cloneable을 구현한다고 추가해야 한다. 
    @Override public PhoneNumber clone() {
        try {
          	// PhoneNumber로 형변환해서 리턴하는 부분 체크
            return (PhoneNumber) super.clone();
          // PhoneNumber는 Cloneable을 구현하고 있으니 catch문을 탈일이 없다.
          // 비검사 예외(unchecked exception)였어야 했다는 신호(아이템 71)
          // 상태 검사 메서드와 비검사 예외를 던지는 메서드로 리팩토링 할 수 있다.(아이템 71 참고)
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();  // 일어날 수 없는 일이다.
        }
    }
}  
```

- 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완벽히 우리가 원하는 상태라 더 손볼 것이 없다.
  - 쓸데없는 복사를 지양한다는 관점에서 보면 불변 클래스는 굳이 `clone` 메서드를 제공하지 않는 게 좋다.
    - 밑에 예제를 보면 `super.clone()` 이렇게 구현함.
- 자바의 공변 반환 타이핑(covariant return typing)
  - 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다.
  - 클라이언트가 형변환하지 않아도 되게끔 객체를 반환하기 전에 `PhoneNumber`로 형변환을 해주고 있다.
- 상위 클래스에 메서드 오버라이드 하고 접근 제한자 바꾸는 부분 체크
- 상위 클래스가 있는 하위 클래스의 생성자에서 필요하다면 상위 클래스의 특정 생성자를 `super`로 호출하는 부분 체크
  - `super.somethiing`로 직접 혹은 `super(something)`으로 생성자를 활용
- `PhoneNumber` 클래스의 `clone()` 메서드를 보면 `return super.clone()`
  - 어떻게 `super`를 썼는데 `PhoneNumber`에 완벽한 복제본이 나온다는 거지?
  - 예전에 내 질문에 동규가 달아준 답변 참고
    - 상위 타입으로 하면 JVM에서 상위타입을 쳐다본다던데 상위타입을 쳐다보는데 재정의한 메서드는 자식의 것을 읽어온다는 것이 이해가 되지 않는다는 질문이었음.
    - [late binding](https://brainbackdoor.tistory.com/75)
      - JVM에 의해서 자동으로 VMI(Virtual Method Invocation)이 일어나서 `Overriding`이 있을 경우 자식의 메소드가 호출된다.
      - 비 객체 지향 언어는 컴파일러가 메소드 호출을 발견하면 정확히 어떤 대상 코드를 호출해야 하는지 결정하는데 객체 지향 언어는 런타입에 그 결정을 내릴 수 있다.
      - `super.clone()` 을 하면 `PhoneNumber`가 `clone()` 메서드를 오버라이드 하고 있기 때문에 `Object`의 `clone()`을 호출하는게 아니라 `PhoneNumber`에서 재정의한 `clone()` 메서드가 호출된다.
    - 코드 상에서는 `super`. 찍고 현재 클래스의 변수에 당연히 접근을 못한다.
      - 런타임에 디버깅 걸고 evaluate expression에 super 치면 상위 클래스의 인스턴스 뿐만 아니라 하위 인스턴스에도 접근 할 수 있다.
      - 코드상에서와 런타임에 디버깅 찍고 접근하는것 둘을 구분을 하자.




### 가변 상태를 참조하는 클래스용 clone 메서드

- 배열을 참조하는 `Stack` 클래스 복제하기
  - `clone` 메서드는 사실상 생성자와 같은 효과를 낸다. 즉 clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야한다. 
    - `Stack`의 `clone` 메서드는 제대로 동작하려면 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 `elements` 배열의 `clone`을 재귀적으로 호출해주는 것이다.(밑에 예제 참고)
  - `elements.clone`의 결과를 `Object[]`로 형변환할 필요는 없다. 
    - 배열의 `clone`은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 
    - 따라서 배열을 복제할 때는 배열의 `clone` 메서드를 사용하라고 권장한다.
    - 배열은 `clone` 기능을 제대로 사용하는 유일한 예라 할 수 있다.
  - `elements` 필드가 `final`이었다면 앞서의 방식은 동작하지 않는다. 
    - `final` 필드에는 새로운 값을 할당할 수없기 때문이다.
    - `Cloneable` 아키텍처는 '가변 객체를 참조하는 필드는 `final`로 선언하라'는 일반 용법과 충돌한다.
      - 단, 원본과 복제된 객체가 그 가변 객체를 공유해도 안전하다면 괜찮다.

```java
public class Stack implements Cloneable {
		private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  	...
      
    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }  
      
    // 코드 13-2 가변 상태를 참조하는 클래스용 clone 메서드
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}  
```



- 해시테이블용 `clone` 메서드

  - `clone`을 재귀적으로 호출하는 것만으로는 충분하지 않다.

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        // 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
        // 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비하여, 리스트가 길면 스택 오버플로를 일으킬 위험이 있다.
        Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy());
        }

        // 엔트리 자신이 가리키는 연결 리스트를 반복적으로 복사한다.
        Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for (Entry p = result; p.next != null; p = p.next)
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            return result;
        }
    }

    @Override public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++)
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
  
    ...
      
}
```



- 상속해서 쓰기 위한 클래스 설계 방식 두 가지(아이템 19) 중 어느 쪽에서든, 상속용 클래스는 `Cloneable`을 구현해서는 안 된다.
  - `Object` 방식을 모방할 수도 있다.
    - 제대로 작동하는 `clone` 메서드를 구현해 `protected`로 두고 `CloneNotSupportedException`도 던질 수 있다고 선언.
  - 다른 방법으로는, `clone`을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 할 수도 있다.
    - 밑에 예제와 같이 clone을 퇴화시켜놓으면 된다.

```java
// 하위 클래스에서 Cloneable을 지원하지 못하게 하는 clone 메서드
@Override
protected final Object clone() throws CloneNotSupportedException {
		throw new  CloneNotSupportedException();
}
```



- `Cloneable`을 구현한 스레드 안전 클래스를 작성할 때는 `clone` 메서드 역시 적절히 동기화해줘야 한다(아이템 78).

- 요약

  >Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다.
  >이때 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경한다.
  >이 메서드는 가장 먼저 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정한다.
  >일반적으로 이 말은 그 객체의 내부 '깊은 구조'에 숨어 있는 모든 가변 객체를 복사하고,
  >복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 함을 뜻한다.
  >이러한 내부 복사는 주로 clone을 재귀적으로 호출해 구현하지만,
  >이 방식이 항상 최선인 것은 아니다.
  >기본 타입 필드와 불편 객체 참조만 갖는 클래스라면 아무 필드도 수정할 필요가 없다.
  >단, 일련번호나 고유 ID는 비록 기본 타입이나 불변일지라도 수정해줘야 한다.	



- `Cloneable`을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 `clone`을 잘 작동하도록 구현해야한다.
  - 그렇지 않은 상황에서는 **복사 생성자**와 **복사 팩터리**라는 더 나은 객체 복사 방식을 제공할 수 있다.
  - 더 정확한 이름은 '**변환 생성자**(conversion constructor)', '**변환 팩터리**(conversion factory)'
- **복사 생성자와 그 변형인 복사 팩터리는 `Cloneable/clone` 방식보다 나은 면이 많다.**
  - 언어 모순적이고 위험천만한 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지 않는다.
  - 엉성하게 문서화된 규약에 기대지 않는다.
  - 정상적인 `final` 필드 용법과도 충돌하지 않으며, 불필요한 검사 예외를 던지지 않고, 형변환도 필요치 않다.
- 복사 생성자와 복사 팩터리는 해당 클래스가 구현한 '인터페이스' 타입의 인스턴스를 인수로 받을 수 있다.
  - 클라이언트는 원본 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.
  - `HashSet` 객체를 `TreeSet` 타입으로 복제할 수 있다.

```java
// 복사 생성자
public Yum(Yum yum) { ... };

// 복사 팩터리
public static Yum newInstance(Yum yum) { ... };
```



## 아이템 14. Comparable을 구현할지 고려하라

> In summary, whenever you implement a value class that has a sensible order- ing, you should have the class implement the Comparable interface so that its instances can be easily sorted, searched, and used in comparison-based collec- tions. When comparing field values in the implementations of the compareTo methods, avoid the use of the < and > operators. Instead, use the static compare methods in the boxed primitive classes or the comparator construction methods in the Comparator interface.


> 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다. compareTo 메서드에서 필드의 값을 비교할 때 < 와 > 연산자는 쓰지 말아야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.



- `compareTo`는 단순 동치성 비교에 더해 **순서까지 비교**할 수 있으며, 제네릭하다.
- `Comparable`을 구현했다는 것은 그 클래스의 인스턴스들에는 **자연적인 순서(natural order)**가 있음을 뜻한다.
- 밑에 예제를 보면 `String`이 `Comparable`을 구현 했기 때문에 중복을 제거하고 알파벳순으로 출력이 된다.

```java
// Comparable 구현 시의 이점
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```



- 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 `Comparable` 인터페이스를 구현하자.
  - `Comparable`을 구현하면 이 인터페이스를 활용하는 수많은 제네릭 알고리즘과 컬렉션의 힘을 누릴 수 있다.
  - 사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입(아이템 34)이 `Comparable`을 구현했다.
- 모든 객체에 대해 전역 동치관계를 부여하는 `equals` 메서드와 달리, `compareTo`는 **타입이 다른 객체**를 신경 쓰지 않아도 된다.

- 비교를 활용하는 클래스의 예
  - 정렬된 컬렉션인 `TreeSet`과 `TreeMap`
  - 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스인 `Collections`와 `Arrays`가 있다.
- `compareTo` 규약 자세히 살펴보자.
  - 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.
    - 첫 번째가 두 번째보다 크면, 두 번째는 첫 번째보다 작아야 한다.
  - 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 한다.
  - 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
    - `compareTo` 메서드로 수행한 동치성 테스트의 결과가 `equals`와 같아야 한다.
    - `Collection`, `Set`, `Map` 인터페이스들은 `equals` 메서드의 규약을 따르다고 되어 있지만, 놀랍게도 **정렬된 컬렉션들은 동치성을 비교할 때 equals 대신 compareTo를 사용**하기 때문에 아주 큰 문제는 아니지만, 주의해야 한다.
      - `new BigDecimal("1.0")`, `new BigDecimal("1.00")`
        - `HashSet`에 넣으면 `equals` 메서드로 비교해서 원소 2개를 가진다.
        - `TreeSet`에 넣으면 `compareTo` 메서드로 비교해서 원소 1개를 가진다.
          - `Tree`는 정렬된 컬렉션

- `Comparable`을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독립된 클래스를 만들고, 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드를 두자.
  - 바깥 클래스에 우리가 원하는 `compareTo` 메서드를 구현해 넣으면 된다.

- `compareTo` 메서드 작성 요령은 `equals`와 비슷하다. 몇 가지 차이점만 주의하자.
- `Comparable`은 **타입을 인수로 받는 제네릭 인터페이스**이므로 `compareTo` 메서드의 인수 타입은 컴파일타임에 정해진다.
  - 인수의 타입이 잘못됐다면 컴파일 자체가 되지 않는다.
  - `Comparable<PhoneNumber>` - `compareTo` 메서드 인자값의 타입은 `PhoneNumber`

- `compareTo` 메서드는 각 필드가 동치인지를 비교하는 게 아니라 그 순서를 비교한다.
  - 객체 참조 필드를 비교하려면 `compareTo` 메서드를 재귀적으로 호출한다.
  - `Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(`Comparator`)를 대신 사용한다.

```java
// 코드 14-1 객체 참조 필드가 하나뿐인 비교자
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 수정된 equals 메서드
    @Override
    public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }

    @Override
    public int hashCode() {
        return s.hashCode();
    }

    @Override
    public String toString() {
        return s;
    }

    // 자바가 제공하는 비교자를 사용해 클래스를 비교한다.
  	// Comparator<String> CASE_INSENSITIVE_ORDER = new CaseInsensitiveComparator();
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
}
```



- `compareTo` 메서드에서 관계 연산자 `<` 와 `>`를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니, 이제는 추천하지 않는다.
  - 박싱된 기본 타입 클래스들에 새로 추가된 **정적 메서드인 `compare`**를 이용하면 되는 것이다.
    - `Short.compare()`

- 클래스에 핵심 필드가 여러 개라면 어느 것을 먼저 비교하느냐가 중요해진다. 
  - 가장 핵심적인 필드부터 비교해나가자.
  - 핵심적인 필드의 비교에서 결과가 나오면 바로 결과를 반환하자.
  - 가장 핵심이 되는 필드가 똑같다면, 똑같지 않은 필드를 찾을 때까지 그다음으로 중요한 필드를 비교해나간다.

```java
// 코드 14-2 기본 타입 필드가 여럿일 때의 비교자
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(areaCode, pn.areaCode);
  if (result == 0)  {
    result = Short.compare(prefix, pn.prefix);
    if (result == 0)
      result = Short.compare(lineNum, pn.lineNum);
  }
  return result;
}
```



- 자바 8에서는 `Comparator` 인터페이스가 일련의 비교자 생성 메서드(comparator construction method)와 팀을 꾸려 **메서드 연쇄 방식**으로 비교자를 생성할 수 있게 되었다.
  - 이 비교자들을 `Comparable` 인터페이스가 원하는 `compareTo` 메서드를 구현하는 데 멋지게 활용할 수 있다.
  - 하지만 약간의 성능 저하가 뒤따른다는 점.

```java
// 코드 14-3 비교자 생성 메서드를 활용한 비교자
private static final Comparator<PhoneNumber> COMPARATOR =
  // 입력 인수의 타입(PhoneNumber pn)을 명시
  // 자바의 타입 추론 능력이 이 상황에서 타입을 알아낼 만큼 강력하지 않다.
  comparingInt((PhoneNumber pn) -> pn.areaCode)
  // 이번에는 타입을 명시하지 않았다.
  // 자바의 타입 추론 능력이 이 정도는 추론해낼 수 있다.
  .thenComparingInt(pn -> pn.prefix)
  .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
  return COMPARATOR.compare(this, pn);
}
```



- `Comparator`는 수많은 보조 생성 메서드들로 중무장하고 있다.
  - `long`과 `double`용으로는 `comparingInt`와 `thenComparingInt`의 변형 메서드를 준비했다.
    - `short`처럼 더 작은 정수 타입에는 `int`용 버전을 사용하면 된다.
    - `float`는 `double`용 버전을 이용해 수행
  - 객체 참조용 비교자 생성 메서드도 준비되어 있다.
- 이따금 '값의 차'를 기준으로 첫 번째 값이 두 번째 값보다 작으면 음수를, 두 값이 같으면 0을, 첫 번째 값이 크면 양수를 반환하는 `compareTo`나 `compare` 메서드와 마주할 것이다.
  - 코드 14-4 처럼 직접 하면 안된다!
  - **정적 compare 메서드나 비교자 생성 메서드를 활용하자.**

```java
// 코드 14-4 해시코드 값의 차를 기준으로 하는 비교자 - 추이성을 위배한다!
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return o1.hashCode() - o2.hashCode();
  }
};

// 코드 14-5 정적 compare 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return Integer.compare(o1.hashCode(), o2.hashCode());
  }
};

// 코드 14-6 비교자 생성 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = 
  	Comparator.comparingInt(o -> o.hashCode());
```
