

# 아이템 15. 클래스와 멤버의 접근 권한을 최소화하라

> To summarize, you should reduce accessibility of program elements as much as possible (within reason). After carefully designing a minimal public API, you should prevent any stray classes, interfaces, or members from becoming part of the API. With the exception of public static final fields, which serve as constants, public classes should have no public fields. Ensure that objects referenced by public static final fields are immutable.

> 프로그램 요소의 접근성은 가능한 한 최소한으로 하라. 꼭 필요한 것만 골라 최소한의 public API를 설계하자. 그 외에는 클래스, 인터페이스, 멤버가 의도치 않게 API로 공개되는 일이 없도록 해야 한다. public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져서는 안 된다. public static final 필드가 참조하는 객체가 불변인지 확인하라.



- 어설프게 설계된 컴포넌트와 잘 설계된 컴포넌트의 가장 큰 차이
  - 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼느냐.
    - 잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다.



### 정보 은닉의 장점

* 시스템 개발 속도를 높인다.
  * 여러 컴포넌트를 병렬로 개발할 수 있기 때문 (의존하는 모듈의 인터페이스(API)만 알면 된다.) 

* 시스템 관리 비용을 낮춘다.
  * 각 컴포넌트가 의존관계가 얕으니 더 빨리 파악 할 수 있다.
  * 서로가 API로만 소통하니 구현체를 바꾸기 용이하다.

* 정보 은닉 자체가 성능을 높여주지는 않지만, 성능 최적화에 도움을 준다.
  * 다른 컴포넌트에 영향을 주지 않고 원하는 컴포넌트만 최적화 할 수 있다. (A 컴포넌트를 수정해도 B 컴포넌트가 영향을 받지 않는다.)
* 소프트웨어 재사용성을 높인다.
  * 외부에 거의 의존하지 않고 독자적으로 동작할 수 있는 컴포넌트라면 낮선 환경에서도 재사용 용이
* 큰 시스템을 제작하는 난이도를 낮춰준다.
  * 전체가 완성되지 않아도 개별 컴포넌트의 동작을 검증 할 수 있다.



### 접근성을 가능한 좁히기

- **접근 제한자를 제대로 활용**하는 것이 **정보 은닉의 핵심**
- 기본 원칙은 모든 클래스와 멤버의 접근성을 **가능한 한 좁혀야** 한다는 것이다.

- 톱레벨 클래스(일반 클래스)와 인터페이스에 부여 할 수 있는 접근 수준은 `package-private`, `public` 두 가지다.
  - 보통 `package-private`을 `default` 라고 불렀었다.
    - 같은 패키지 안에서 쓸 수 있고 + 해당 클래스 안에서만 쓸 수 있는 `private`  그래서 이름이 `package-private`인가 보다.
  - `public`으로 선언하면 공개 API가 되고 `default`로 선언하면 패키지 안에서만 사용할 수 있다.




- 그냥 습관적으로 접근 수준을 `public`으로 주지말고 패키지 외부에서 쓸 이유가 없다면 `package-private`로 선언하자. 
  - 한 클래스에서만 사용하는 클래스나 인터페이스가 있다면 그 사용하는 클래스 안에 `private static class`로 중첩시켜보자.

  - `public` 일 필요가 없는 클래스의 접근 수준을` package-private` 톱레벨 클래스로 좁히자. 
    - `package-private` 톱 레벨 클래스는 내부 구현에 속하기 때문에 API가 공개되고 난 뒤에도 내부 구현을 언제든지 수정할 수 있다. 
      - jar 안에 `java.util.ArrayPrefixHelpers` class의 경우 접근 수준이 `package-private`



- 클래스의 공개 API는 모든 멤버 변수 우선 `private`으로 묶고 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한하여 `private` 제한자를` package-private`으로 풀어주자. 
  - 이런 작업이 잦다면 시스템에서 컴포넌트를 더 분해해야 하는 것은 아닌지 다시 고민해보자. 
  - `Serializable`을 구현한 클래스에서는 그 필드들도 의도치 않게 공개 API가 될 수도 있다(아이템 86, 87).

- `public` 클래스에서 멤버의 접근 수준을 `package-private`에서` protected`로 변경하는 순간 그 멤버에 접근할 수 있는 대상 범위가 엄청나게 넓어진다. 
  - **`public` 클래스의 `protected` 멤버는 공개 API**이므로 영원히 지원돼야 한다. 
  - 그러니 `protected` 멤버의 수는 적을수록 좋다.



- 상위 클래스의 메서드를 재정의할 때는 그 접근 수준을 상위 클래스에서 보다 **좁게** 설정 할 수 없다. 
  - 이 제약은 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다는 규칙(**리스코프 치환 원칙**, 아이템 10)을 지키기 위해 필요하다.
  - 클래스가 인터페이스를 구현하는 건 이 규칙의 특별한 예로 볼 수 있고, 이때 클래스는 인터페이스가 정의한 모든 메서드를 `public`으로 선언해야 한다.
- 테스트를 위해서 클래스의 `private` 멤버를 `package-private`까지 풀어주는 것은 허용할 수 있지만, 그 이상은 안된다. 



- `public` 클래스의 인스턴스 필드는 되도록 **`public`이 아니어야 한다**(아이템 16).
  - 필드가 **가변 객체를 참조**하거나, **`final`이 아닌 인스턴스 필드를 `public`으로 선언**하면 그 필드와 관련된 모든 것은 **불변식**을 보장할 수 없게 된다. 
    - 필드가 수정될 때 (락 획득 같은) 다른 작업을 할 수 없게 되므로 **`public` 가변 필드를 갖는 클래스**는 일반적으로 **스레드** 안전하지 않다.
      - 이 부분은 다시
  - **필드가 `final`이면서 불변 객체를 참조**하더라도 문제는 여전히 남는데 내부 구현을 바꾸고 싶어도 그 `public` 필드를 없애는 방식으로는 리펙터링할 수 없게 된다.
  - 이러한 문제는 정적 필드에서도 마찬가지이나 예외가 하나 있는데 꼭 필요한 경우의 상수는 `public static final` 필드로 공개해도 좋다.
    - 이런 필드는 반드시 **기본 타입이나 불변 객체를 참조**해야 한다(아이템 17).



- 길이가 0이 아닌 배열은 모두 변경 가능하니 주의하자. 
  - 테스트 코드를 만들어서 실습해 보니 final이지만 변경이 되는군.
  - 따라서 클래스에서 `public static final` 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다. 
  - 해결책은 두가지다.
    -  public 배열을 private으로 만들고 public 불변 리스트를 추가
    -  배열을 private으로 만들고 그 복사본을 반환하는 public 메서드를 추가하는 방법(방어적 복사).
  - 어떤 반환 타입이 더 쓰기 편한지, 성능은 어느 쪽이 더 나은지 등을 고민해서 정하자.

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUE));
```

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values(){
    return PRIVATE_VALUES.clone();
}
```



### 암묵적 접근 수준

- 패키지가 클래스들의 묶음이듯, 모듈은 패키지들의 묶음이다.

- 자바 9에서는 모듈 시스템이라는 개념이 도입되면서 두 가지 암묵적 접근 수준이 추가되었다.
  - 모듈은 자신에 속하는 패키지 중 공개(export)할 것들을 선언한다.
    - `protected` 혹은 `public` 멤버라도 해당 패키지를 공개하지 않았다면 모듈 외부에서는 접근할 수 없다.
  - 모듈 시스템을 활용하면 클래스를 외부에 공개하지 않으면서도 같은 모듈을 이루는 패키지 사이에서는 자유롭게 공유할 수 있다.
- 이 암묵적 접근 수준들은 각각 public 수준과 protected 수준과 같으나 **그 효과가 모듈 내부로 한정**되는 변종인 것이다.
- 모듈의 JAR 파일을 자신의 모듈 경로가 아닌 애플리케이션의 클래스패스(classpath)에 두면 그 모듈 안의 모든 패키지는 마치 모듈이 없는 것 처럼 행동한다.
- 새로 등장한 이 접근 수준을 적극 활용한 대표적인 예가 바로 JDK 자체다.
  - 자바 라이브러리에서 공개하지 않은 패키지들은 해당 모듈 밖에서는 절대 접근할 수 없다.



# 아이템 16. public 클래스 안에는 public 필드를 두지 말고 접근자 메서드를 사용하라

> In summary, public classes should never expose mutable fields. It is less harmful, though still questionable, for public classes to expose immutable fields. It is, however, sometimes desirable for package-private or private nested classes to expose fields, whether mutable or immutable.

> public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.



- 코드 16-1 예제에 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다(아이템 15).
  - API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
    - 벌써 공개 되었으니껜
  - 불변식을 보장할 수 없다.
  - 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다.
    - Setter에 뭔가 작업을 해두는걸 말하는 듯.
  - 필드를 모두 `private`으로 바꾸고 `public` 접근자(getter)를 추가한다.

```java
// 코드 16-1 이처럼 퇴보한 클래스는 public이어서는 안 된다!
class Point {

  public double x;
  public double y;
  
}
```



- 패키지 바깥에서 접근할 수 있는 클래스(public 클래스)라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.
  - `public` 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 없게된다.
  - 이미 제공한 접근자만 유지를 해주면 내부 표현 방식을 언제든지 바꿀 수 있다 이런 뜻인듯. 

```java
// 코드 16-2 접근자와 변경자(mutator) 메서드를 활용해 데이터를 캡슐화한다.
class Point {
  
  private double x;
  private double y;
  
  public Point(double x, double y){
    this.x = x;
    this.y = y;
  }
  
  public double getX() { return x; }
  public double getY() { return y; }
  
  public void setX( double x ) { this.x = x; }
  public void setX( double y ) { this.y = y; }
  
}
```



- `package-private` 클래스 혹은 `private` 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.
  - 클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다. 따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다.
    - 여기서 말하는 클라이언트는 같은 패키지나 같은 클래스 안에서 `package-private` 클래스 혹은 `private` 중첩 클래스를 사용하는 클래스를 얘기하는거 같다.



- 자바 플랫폼 라이브러리에도 `public` 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있다.
  - 대표적인 예가 `java.awt.package` 패키지의 `Point`와 `Dimension` 클래스
    - 내부를 노출한 `Dimension` 클래스의 심각한 성능 문제는 오늘날까지도 해결되지 못했다.
    - 이 클래스들을 흉내 내지 말고, 타산지석으로 삼길 바란다.



- `public` 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어 들지만, 여전히 결코 좋은 생각이 아니다.
  - API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다. 



# 아이템 17. 변경 가능성을 최소화하라

- **불변 클래스**란 간단히 말해 그 인스턴스의 내부 값을 수정할 수 없는 클래스다.
  - 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다.
  - 자바 라이브러리에도 다양한 불변 클래스가 있는데 `String`, 기본 타입의 박싱 클래스들(래퍼 클래스, `Byte`, `Integer` 등), `BigInteger`, `BigDecimal` 이 여기 속한다.
- 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다.



### 클래스를 불변으로 만드는 다섯 가지 규칙

- 객체 상태를 변경하는 메서드(변경자)를 제공하지 않는다. 
- 클래스를 확장할 수 없도록 한다.
- 모든 필드를 `final`로 선언한다.
- 모든 필드를 `private`으로 선언한다.
- 자신 외에 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
  - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다.
  - 생성자, 접근자, readObject 메서드(아이템 88) 모두에서 **방어적 복사**를 수행하라.



- 함수형 프로그래밍
  - 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴
- 절차적 혹은 명령형 프로그래밍
  - 메서드에서 피연산자인 자신을 수정해 자신의 상태가 변하게 된다.

- 함수형 프로그래밍 방식으로 프로그래밍하면 코드에서 불변이 되는 영역의 비율이 높아지는 장점을 누릴 수 있다.



### 불변 객체 장단점, 가변 객체와 비교

- **불변 객체는 단순하다.**
  - 불변 객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.
    - 모든 생성자가 클래스 불변식(class invariant)을 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다.
- 반면 **가변 객체**는 임의의 복잡한 상태에 놓일 수 있다.
  - 변경자 메서드가 일으키는 상태 전이를 정밀하게 문서로 남겨놓지 않은 가변 클래스는 믿고 사용하기 어려울 수도 있다.

- 불변 객체는 근본적으로 **스레드 안전**하여 따로 **동기화**할 필요 없다.
  - 여러 스레드가 동시에 사용해도 절대 훼손되지 않는다.
  - 클래스를 스레드 안전하게 만드는 가장 쉬운 방법
  - 불변 객체에 대해서는 그 어떤 스레드도 다른 스레드에 영향을 줄 수 없으니 불변 객체는 안심하고 공유할 수 있다.

```java
// 코드 17-1 불변 복소수 클래스
// 불변을 설명하기 위한 예로 든 것일 뿐, 실무에서 쓸 만한 수준은 못 된다고
public final class Complex {
    private final double re;
    private final double im;

  	// 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하기를 권한다.
    // 가장 쉬운 재활용 방법은 자주 쓰이는 값들을 상수(public static final)로 제공하는 것
    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

  	// 사칙연산 메서드들이 인스턴스 자신은 수정하지 않고 
  	// 새로운 Complex 인스턴스를 만들어 반환하는 모습에 주목
  
  	// 메서드 이름에 동사 대신 전치사를 사용해서 
    // 메서드가 객체의 값을 변경하지 않는다는 사실을 강조
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```



- 불변 클래스는 자주 사용되는 인스턴스를 **캐싱**하여 같은 인스턴스를 중복 생성하지 않게 해주는 **정적 팩터리**(아이템 1)를 제공할 수 있다.
  - 박싱된 기본 타입 클래스 전부와 `BigInteger`가 여기 속한다.
    - 기본 타입은 다 메모리에 올라가 있고 박싱된 기본 타입이나 `BigInteger` 등도 각자의 default cache size 만큼 메모리에 값을 캐싱하고 있다.
  - 여러 클라이언트가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 줄어든다.
  - 새로운 클래스를 설계할 때 public 생성자 대신 정적 팩터리를 만들어두면, 클라이언트를 수정하지 않고도 필요에 따라 캐시 기능을 나중에 추가할 수 있다.

```java
// BigInteger가 제공하는 정적 팩터리 메서드 valueOf
public class BigInteger extends Number implements Comparable<BigInteger> {
    
		...  
  
    // static 메서드라서 BigInteger.valueOf(); 이렇게 바로 쓸 수 있다.  
		public static BigInteger valueOf(long val) {
        if (val == 0)
            return ZERO;
        if (val > 0 && val <= MAX_CONSTANT)
            return posConst[(int) val];
        else if (val < 0 && val >= -MAX_CONSTANT)
            return negConst[(int) -val];

        return new BigInteger(val);
    }

		...
  
    private static BigInteger posConst[] = new BigInteger[MAX_CONSTANT+1];
    private static BigInteger negConst[] = new BigInteger[MAX_CONSTANT+1];

 		...
    
}    
```



- 자유롭게 공유할 수 있으니 **방어적 복사**(아이템 50)도 필요 없다.
  - 불변 클래스는 clone 메서드나 복사 생성자(아이템 13)를 제공하지 않는 게 좋다.
  - `String` 클래스의 복사 생성자는 나쁜예, 되도록 사용하지 말아야 한다(아이템 6). 



- 불변 객체는 자유롭게 공유할 수 있고 불변 객체끼리는 **내부 데이터를 공유**할 수 있다.
  - `BigInteger` 클래스는 내부에서 값의 부호(sing)와 크기(magnitude)를 따로 표현
    - 부호에는 `int` 변수를, 크기(절댓값)에는 `int` 배열을 사용
  - negate 메서드는 크기가 같고 부호만 반대인 새로운 `BigInteger`를 생성해준다.
    - 이때 배열은 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유
      - 새로운 `BigInteger`를 생성하지만 부호만 바꾸고 배열은 공유하는거다.



- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
  - 값이 바뀌지 않는 구성요소들로 이뤄진 객체라면 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하기 때문이다.
    - 좋은 예로, 불변 객체는 맵의 키와 집합(`Set`)의 원소로 쓰기에 안성맞춤이다.



- 불변 객체는 그 자체로 **실패 원자성**을 제공한다(아이템 76).
  - 상태가 절대 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다.



- 불변 클래스에 단점으로는 **값이 다르면 반드시 독립된 객체로** 만들어야 한다는 것이다. 
  - 백만 비트짜리 `BigInteger`에서 비트 하나를 바꿔야 한다고 해보자.
    - 원본과 단지 한 비트만 다른 백만 비트짜리 인스턴스를 생성해야 한다.
      - filipBit 메서드는 새로운 `BigInteger` 인스턴스를 생산한다.
        - 이 연산은 `BigInteger`의 크기에 비례해 시간과 공간을 잡아먹는다.
        - 성능 문제 이슈가 발생할 수 있다.



- 클래스가 불변임을 보장하려면 자신을 상속하지 못하게 해야 한다.
  - 자신을 상속하지 못하게 하는 가장 쉬운 방법은 `final` 클래스로 선언하는 것
  - 더 유연한 방법으로는 모든 생성자를 `private` 혹은 `package-private`으로 만들고 `public` 정적 팩터리를 제공하는 방법이다(아이템 1).
    - 정적 팩터리 방식은 다수의 구현 클래스를 활용한 유연성을 제공하고, 이에 더해 다음 릴리스에서 객체 캐싱 기능을 추가해 성능을 끌어올릴 수도 있다.



### 가변 동반 객체

- `MutableBigInteger`와 `SignedMutableBigInteger`처럼 `BigInteger`와 거의 같은 기능을 하지만 **가변 객체**를 생성하는 클래스를 **동반 클래스**라고 한다.
  - 프로그래머들이 직접 이 동반 클래스들을 다루는 것은 매우 어려운 일이지만, 이미 BigInteger 클래스가 동반 클래스들을 적절히 써서 어려운 계산을 하기 때문에 우리는 `BigInteger` 클래스를 쓰기만 하면된다. 
  - 가변(mutable) 객체는 생성 후에도 상태를 변경할 수 있는 객체임.
  - **가변 동반 객체**
    - **불변 객체와 거의 같은 기능을 하는데 가변 객체를 생성하는 클래스**
  - 하지만 `package-private`의 가변 동반 클래스는 클라이언트들이 원하는 복잡한 연산들을 정확히 예측할 수 있을 때 쓸 수 있다.
    - `MutableBigInteger`와 `SignedMutableBigInteger`는 둘다 접근 수준은 `package-private`임.
    - 그렇지 않다면 이 클래스를 public으로 제공하는 게 최선이다.
      - 자바 플랫폼 라이브러리에서 이에 해당하는 대표적인 예가 `String` 클래스다.
        - `String`의 가변 동반 클래스는 `StringBuilder`(그리고 전임자 `StringBuffer`)  
          - `StringBuilder`는 `final class` 이지만, 할당된 값을 변경하면 새로운 객체를 만드는 방식이 아니라 기존 할당된 값을 수정하는 것으로 처리하기에 가변객체임.
            - 클래스에 `final`은 상속을 못하게 하니깐.



### 정리

- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
  - 게터(getter)가 있다고 해서 무조건 세터(setter)를 만들지는 말자.
  - 불변 클래스는 장점이 많으며, 단점이라곤 특정 상황에서의 잠재적 성능 저하뿐이다.
  - `PhoneNumber`와 `Complex`같은 단순한 값 객체는 항상 불변으로 만들자.
  - `String`과 `BigInteger`처럼 무거운 값 객체도 불변으로 만들 수 있는지 고심해야 한다.
    - 성능 때문에 어쩔 수 없다면(아이템 67) 불변 클래스와 쌍을 이루는 가변 동반 클래스를 `public` 클래스로 제공하도록 하자.



- 모든 클래스를 불변으로 만들 수는 없겠지만, 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
  - 객체가 가질 수 있는 상태의 수를 줄이면 그 객체를 예측하기 쉬워지고 오류가 생길 가능성이 줄어든다.
  - 꼭 변경해야 할 필드를 뺀 나머지 모두를 `final`로 선언하자.
  - 이번 아이템과 아이템 15의 조언을 종합하면
    - **다른 합당한 이유가 없다면 모든 필드는 `private final`이어야 한다.**



- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.

  - 확실한 이유가 없다면 생성자의 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공해서는 안 된다.
  - 객체를 재활용할 목적으로 상태를 다시 초기화하는 메서드도 안 된다.
  - 복잡성만 커지고 성능 이점은 거의 없다.

  

- `java.util.concurrent` 패키지의 `CountDownLatch` 클래스가 이상의 원칙을 잘 방증한다.

  - 비록 가변 클래스지만 가질 수 있는 상태의 수가 많지 않다.
  - 인스턴스를 생성해 한 번 사용하고 그걸로 끝이다.
  - 카운트가 0에 도달하면 더는 재사용할 수 없다.



### 기타. static 초기화 블럭

- 테스트 코드에서 `BigInteger` valueOf 메서드 디버깅 찍어서 따라가는데 static 초기화 블럭으로 빠지던데 어떻게 돌아가는걸까
  - 자바 기본 다시 정리
    - 변수 종류
      - `int iv;` // 인스턴스 변수
      - `static int cv;` // 클래스 변수
      - 메서드 안에 `int lv;` // 지역 변수
    - 멤버변수(cv, iv)의 초기화
      - 명시적 초기화(`=`)
      - 초기화 블럭
        - **인스턴스 초기화 블럭**
          - `{}`
        - **클래스 초기화 블럭**
          - `static {}`
      - 생성자(iv 초기화)
  - 다시 질문으로 돌아가서 static 메서드가 호출 될 때 `static` 초기화 블럭이 호출이 될까?
    - 호출된다.
    - 디버깅을 `static` 라인에 걸지말고 `{}`안에 걸자.
    - `static` 초기화 블럭에서 static 변수를 초기화 하게 해놓고 외부에서 `static` 변수를 호출 하면 `static` 초기화 블럭 호출 되는지도 확인하자. 
      - 당연히 호출된다.
  - `BigInteger`에 static 메서드인 valueOf가 호출되면 클래스 초기화 블럭도 호출이 되고 거기에서 캐싱 관련된 작업을 하고있다.

```java
public class StaticInitializationBlock {

    private static String a;
    static String b;

    static {
        a = "상코";
        b = "SANGCO";
    }

    public static BigInteger valueOf(String val) {
        return new BigInteger(val);
    }

    public static void main(String[] args) {
        valueOf("15");
        System.out.println(a);
        System.out.println(b);

        BigInteger bi = BigInteger.valueOf(0);
        System.out.println(bi);
    }

}
```



## 아이템 18. 상속보다는 컴포지션을 사용하라

> To summarize, inheritance is powerful, but it is problematic because it violates encapsulation. It is appropriate only when a genuine subtype relationship exists between the subclass and the superclass. Even then, inheritance may lead to fragility if the subclass is in a different package from the superclass and the superclass is not designed for inheritance. To avoid this fragility, use composition and forwarding instead of inheritance, especially if an appropriate interface to implement a wrapper class exists. Not only are wrapper classes more robust than subclasses, they are also more powerful.

> 상속은 강력하지만 캡슐화를 해친다는 문제가 있다. 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다. is-a 관계일 때도 안심할 수만은 없는 게, 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다. 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용하자. 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다. 래퍼 클래스는 하위클래스보다 견고하고 강력하다.



- 상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다.
  - 상위 클래스와 하위 클래스를 모두 같은 프로그래머가 통제하는 패키지 안에서라면 상속도 안전한 방법이다.
  - 확장할 목적으로 설계되었고 문서화도 잘 된 클래스(아이템 19)도 마찬가지로 안전하다.
  - **다른 패키지의 구체 클래스를 상속하는 일은 위험하다.**
    - 이번 아이템의 '상속'은 **클래스가 다른 클래스를 확장하는 구현 상속**을 말한다.
      - 클래스가 인터페이스를 구현하거나 인터페이스가 다른 인터페이스를 확장하는 **인터페이스 상속**과는 무관하다.



- 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.
  - 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
  - 여기서 말하는 메서드 호출의 의미는 뒤에 나온다.
    - composition, forwarding, forwarding method



- 상속이 캡슐화를 깨뜨리는 구체적인 예
  - 성능을 높이려고 `HashSet`은 처음 생성된 이후 원소가 몇 개 더해졌는지 알 수 있게 만들려고 한다.
    - 원소가 몇 개 더해졌느냐는 `HashSet`의 현재 크기와는 다른 개념이다.
    - 이 클래스는 잘 구현된 것처럼 보이지만 제대로 작동하지 않는다.

```java
// 코드 18-1 잘못된 예 - 상속을 잘못 사용했다! (114쪽)
public class InstrumentedHashSet<E> extends HashSet<E> {
    
  	// 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
      	// addAll()은 각 원소를 추가할 때 add() 메서드를 호출하는데
      	// 이때 호출되는 add()는 InstrumentedHashSet이 재정의한 add() 메서드이다.
      	// addCount++가 두번 호출 되는거다.
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        // List.of() 자바 9부터 이전 버전은 Arrays.asList()
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
  
}
```



- 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.
  - 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 **컴포지션**(composition; 구성)이라 한다.
- 새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다.
  - 이 방식을 **전달**(forwarding)이라 하며, 새 클래스의 메서들을 **전달 메서드**(forwarding method)라 부른다.
- 이렇게 하면 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.

```java
// 코드 18-2 래퍼 클래스 - 상속 대신 컴포지션을 사용했다.
// 상위 클래스인 ForwardingSet 클래스에서 private final Set<E> s; 필드로 가지고 있다.
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
  
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
      	// 여기서 super.addAll() 하면
      	// ForwardingSet에 addAll()로
        // 거기서 필드로 가지고 있는 Set에 addAll() 호출
        // 그러면 Set 구현체 HashSet의 addAll(), add() 메서드 호출
        return super.addAll(c);
    }
    
  	public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

```java
// 코드 18-3 재사용할 수 있는 전달 클래스
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
    public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
    public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o) { return s.equals(o); }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```



- 상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다.
  - 다르게 말하면, 클래스 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속해야 한다.
- 자바 플랫폼 라이브러리에서 이 원칙을 명백히 위반한 사례
  - 스택은 벡터가 아니므로 `Stack`은 `Vector`를 확장해서는 안 됐다.
  - 속성 목록도 해시테이블이 아니므로 `Properties`도 `Hashtable`을 확장해서는 안 됐다.
- 컴포지션 대신 상속을 사용하기로 결정하기 전에 마지막으로 자문해야 할 질문을 소개한다.
  - 확장하려는 클래스의 API에 아무런 결함이 없는가? 
  - 결함이 있다면, 이 결함이 여러분 클래스의 API까지 전파돼도 괜찮은가?
- 컴포지션으로는 이런 결함을 숨기는 새로운 API를 설계할 수 있지만, 상속은 상위 클래스의 API를 '그 결함까지도' 그대로 승계한다.



## 아이템 19. 상속을 고려해 설계하고 문서화 하라

> In summary, designing a class for inheritance is hard work. You must document all of its self-use patterns, and once you’ve documented them, you must commit to them for the life of the class. If you fail to do this, subclasses may become depen- dent on implementation details of the superclass and may break if the implementa- tion of the superclass changes. To allow others to write *efficient* subclasses, you may also have to export one or more protected methods. Unless you know there is a real need for subclasses, you are probably better off prohibiting inheritance by declaring your class final or ensuring that there are no accessible constructors.

> 상속용 클래스를 설계하기란 결코 만만치 않다. 클래스 내부에서 스스로를 어떻게 사용하는지(자기사용 패턴) 모두 문서로 남겨야 하며, 일단 문서화한 것은 그 클래스가 쓰이는 한 반드시 지켜야 한다. 그러지 않으면 그 내부 구현 방식을 믿고 활용하던 하위 클래스를 오동작하게 만들 수 있다. 다른 이가 효율 좋은 하위 클래스를 만들 수 있도록 일부 메서드를 protected로 제공해야 할 수도 있다. 그러니 클래스를 확장해야 할 명확한 이유가 떠오르지 않으면 상속을 금지하는 편이 나을 것이다. 상속을 금지하려면 클래스를 final로 선언하거나 생성자 모두를 외부에서 접근할 수 없도록 만들면 된다.



- 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.
  - 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야 한다.
  - 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다.
    - '재정의 가능'이란 `public`과 `protected` 메서드 중 `final`이 아닌 모든 메서드를 뜻한다.



```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over the collection looking for the
 * specified element.  If it finds the element, it removes the element
 * from the collection using the iterator's remove method.
 *
 * <p>Note that this implementation throws an
 * <tt>UnsupportedOperationException</tt> if the iterator returned by this
 * collection's iterator method does not implement the <tt>remove</tt>
 * method and this collection contains the specified object.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 */
```

- API 문서의 설명 끝에서 종종 "Implementation Requirements"로 시작하는 절을 볼 수 있다.
  - 메서드 주석에 `@ImplSpec` 태크
  - 위 설명에 따르면 iterator() 메서드를 재정의하면 remove() 메서드의 동작에 영향을 줌을 확실히 알 수 있다.



- "좋은 API 문서란 '어떻게'가 아닌 '무엇'을 하는지를 설명해야 한다."라는 격언과는 대치
  - 상속이 캡슐화를 해치기 때문에 일어나는 안타까운 현실
  - 클래스를 안전하게 상속할 수 있도록 하려면 내부 구현 방식을 설명해야만 한다.
    - 상속만 아니었으면 기술하지 않았을 텐데



- 효율적인 하위 클래스를 큰 어려움 없이 만들 수 있게 하려면 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 `protected` 메서드 형태로 공개해야 할 수도 있다.
  - `java.util.AbstractList`의 removeRange() 메서드
    - `List` 구현체의 최종 사용자는 removeRange() 메서드에 관심이 없다.
    - 그럼에도 이 메서드를 제공한 이유는 단지 하위 클래스에서 부분리스트의 clear() 메서드를 고성능으로 만들기 쉽게 하기 위해서다.



- 상속용 클래스를 설계할 때 어떤 메서드를 `protected`로 노출해야할지는 어떻게 결정할까?
  - 심사숙고해서 잘 예측해본 다음, 실제 하위 클래스를 만들어 시험해보는 것이 최선이다.
  - `protected` 메서드 하나하나가 내부 구현에 해당하므로 그 수는 가능한 한 적어야 한다.
    - 너무 적게 노출해서 상속으로 얻는 이점마저 없애지 않도록 주의해야 한다.



- 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 '유일'하다.
  - 꼭 필요한 `protected` 멤버를 놓쳤다면 하위 클래스를 작성할 때 그 빈자리가 확연히 드러난다.
  - 하위 클래스를 여러 개 만들 때까지 전혀 쓰이지 않는 `protected` 멤버는 사실 `private`이었어야 할 가능성이 크다.
  - 저자의 경험상 이러한 검증에는 하위 클래스 3개 정도가 적당하다.
    - 이 중 하나 이상은 제 3자가 작성해봐야 한다.



- 널리 쓰일 클래스를 상속용으로 설계한다면 여러분이 문서화한 내부 사용 패턴과 `protected` 메서드와 필드를 구현하면서 선택한 결정에 영원히 책임져야 함을 잘 인식해야 한다.
  - 이 결절들이 그 클래스의 성능과 기능에 영원한 족쇄가 될 수 있다.
  - 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.



```java
// 재정의 가능 메서드를 호출하는 생성자 - 따라 하지 말 것!
public class Super {
    // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다.
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
    }
    
}
```

```java
// 생성자에서 호출하는 메서드를 재정의했을 때의 문제를 보여준다.
public final class Sub extends Super {
    // 초기화되지 않은 final 필드. 생성자에서 초기화한다.
    private final Instant instant;

    Sub() {
        instant = Instant.now();
    }

    // 재정의 가능 메서드. 상위 클래스의 생성자가 호출한다.
    @Override public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

- 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.
  - 상위 클래스의 생성자가 Sub 클래스에서 오버라이드 된 overrideMe() 메서드를 호출하면서 `NullPointerException` 발생.
  - `private`, `final`, `static` 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.



- `Cloneable`과 `Serializable` 인터페이스는 상속용 설계의 어려움을 한층 더해준다.
  - 둘 중 하나라도 구현한 클래스를 상속할 수 있게 설계하는 것은 일반적으로 좋지 않은 생각이다.
  - 그 클래스를 확장하려는 프로그래머에게 엄청난 부담을 지우기 때문이다.
  - clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
  - `Serializable`을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면 이 메서드들은 `private`이 아닌 `protected`로 선언해야 한다.
    - `private`으로 선언한다면 하위 클래스에서 무시되기 때문이다.
    - 이 역시 상속을 허용하기 위해 내부 구현을 클래스 API로 공개하는 예 중 하나다.



- 상속용으로 설계하지 않은 클래스는 상속을 금지하자.
  - 상속을 금지하는 방법은 두 가지다.
    - 둘 중 더 쉬운 쪽은 클래스를 `final`로 선언하는 방법이다.
    - 두 번째 선택지는 모든 생성자를 `private`이나 `package-private`으로 선언하고 `public` 정적 팩토리를 만들어주는 방법이다.
      - 정적 팩터리 방법은 내부에서 다양한 하위 클래스를 만들어 쓸 수 있는 유연성을 주며, 이와 관련해서는 아이템 17에서 다뤘다.
  - 둘 중 어느 방식이든 좋다.



- 구체 클래스가 표준 인터페이스를 구현하지 않았는데 상속을 금지하면 사용하기에 상당히 불편해진다.
  - 이런 클래스라도 상속을 꼭 허용해야겠다면 클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 이 사실을 문서로 남기자.
    - 재정의 가능 메서드를 호출하는 자기 사용 코드를 완벽히 제거하라는 뜻임.
    - 이렇게 하면 상속해도 그리 위험하지 않은 클래스를 만들 수 있다.
    - 메서드를 재정의해도 다른 메서드의 동작에 아무런 영향을 주지 않기 때문이다.



- 클래스의 동작을 유지하면서 재정의 가능 메서드를 사용하는 코드를 제거할 수 있는 기계적인 방법을 소개한다.
  - 먼저 각각의 재정의 가능 메서드는 자신의 본문 코드를 `private` '도우미 메서드'로 옮기고 이 도우미 메서드를 호출하도록 수정한다.
  - 그런 다음 재정의 가능 메서드를 호출하는 다른 코드들도 모두 이 도우미 메서드를 직접 호출하도록 수정하면 된다.



## 아이템 20. 추상 클래스보다는 인터페이스를 우선하라

> To summarize, an interface is generally the best way to define a type that permits multiple implementations. If you export a nontrivial interface, you should strongly consider providing a skeletal implementation to go with it. To the extent possible, you should provide the skeletal implementation via default methods on the interface so that all implementors of the interface can make use of it. That said, restrictions on interfaces typically mandate that a skeletal implementation take the form of an abstract class.

> 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다. 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려해보자. 골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다. '가능한 한'이라고 한 이유는, 인터페이스에 걸려 있는 구현상의 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 때문이다.



- 자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상 클래스, 이렇게 두가지다.
  - 자바 8부터 인터페이스도 디폴트 메서드를 제공할 수 있게 되었다.
- 둘의 가장 큰 차이는 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다.
  - 자바는 단일 상속만 지원하니, 추상 클래스 방식은 새로운 타입을 정의하는 데 커다란 제약이 될 수 있다.
  - 반면 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.
- 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.
  - 인터페이스가 요구하는 메서드를 (아직 없다면) 추가하고, 클래스 선언에 `implements` 구문만 추가하면 끝이다.
  - 반면 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어려운 게 일반적이다.



- 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다.
  - 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
  - 예를들어 `Comparable`은 자신을 구현한 클래스의 인스턴스들끼리는 순서를 정할 수 있다고 선언하는 믹스인 인터페이스다.
  - 이처럼 대상 타입의 주된 기능에 선택적 기능을 '혼합(mixed in)'한다고 해서 믹스인이라 부른다.



- 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
  - 타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있지만, 현실에는 계층을 엄격히 구분하기 어려운 개념도 있다.
    - 작곡도 하는 가수도 있잖어(밑에 예제 참고).
  - 밑에 예제 정도의 유연성이 항상 필요하지는 않지만, 이렇게 만들어둔 인터페이스가 결정적인 도움을 줄 수도 있다.
    - 같은 구조를 클래스로 만들려면 가능한 조합 전부를 각각의 클래스로 정의한 고도비만 계층구조가 만들어질 것이다.
      - 속성이 n개라면 지원해야 할 조합의 수는 2^n개나 된다.
        - 흔히 조합 폭발(combinational explosion)이라 부르는 현상

```java
public interface Singer {
  	AudioClip sing(Song s);
}

public interface Songwriter {
  	Song compose(int chartPosition);
}

// Singer와 Songwriter 모두를 확장하고 새로운 메서드까지 추가한 제3의 인터페이스를 정의
public interface SingerSongwriter extends Singer, Songwriter {
  	AudioClip strum();
	  void actSensitive();
}
```



- 래퍼 클래스 관용구(아이템 18)와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.

  - 추상 클래스 타입을 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기는 더 쉽다.

- 인터페이스의 메서드 중 구현 방법이 명백한 것은 구현을 디폴트 메서드로 제공해 프로그래머들의 일감을 덜어줄 수 있다.

  - 디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 `@implSpec` 자바독 태그를 붙여 문서화해야 한다(아이템 19).

- 디폴트 메서드에도 제약은 있다.

  - 많은 인터페이스가 `equals`와 `hashCode` 같은 `Object`의 메서드를 정의하고 있지만, 이들은 디폴트 메서드로 제공해서는 안 된다.
  - 인터페이스는 인스턴스 필드를 가질 수 없고 `public`이 아닌 정적 멤버도 가질 수 없다(단, `private` 정적 메서드는 예외다).
  - 내가 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.

- 인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법도 있다.

  - 인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공
  - 골격 구현 클래스는 나머지 메서드들까지 구현
  - 이렇게 해두면 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는 데 필요한 일이 대부분 완료된다.
    - 템플릿 메서드 패턴

- 관례상 인터페이스 이름이 `Interface`라면 그 골격 구현 클래스의 이름은 AbstractInterface로 짓는다.

  - 좋은 예로, 컬렉션 프레임워크의 `AbstractCollection`, `AbstractSet`, `AbstractList`, `AbstractMap` 각각이 바로 핵심 컬렉션 인터페이스의 골격 구현이다.
    - 어쩌면 SkeletalCollection 형태가 더 적절했을지 모르지만 이미 Abstract 접두어가 확고히 자리를 잡았다.

- 골격 구현 클래스의 아름다움은 추상 클래스처럼 구현을 도와주는 동시에, 추사아 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유롭다는 점에 있다.

  - 골격 구현을 확장하는 것으로 인터페이스 구현이 거의 끝나지만, 꼭 이렇게 해야 하는 것은 아니다.
  - 구조상 골격 구현을 확장하지 못하는 처지라면 인터페이스를 직접 구현해야 한다.
    - 이런 경우라도 인터페이스가 직접 제공하는 디폴트 메서드의 이점을 여전히 누릴 수 있다.

- 골격 구현 클래스를 우회적으로 이용할 수도 있다.

  - 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 `private` 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 방식
    - 아이템 18에서 다룬 래퍼 클래스와 비슷한 이 방식을 시뮬레이트한 다중 상속(simulated multiple inheritance)이라 한다.
      - 다중 상속의 많은 장점을 제공하는 동시에 단점은 피하게 해준다.

- 골격 구현 작성은 상대적으로 쉽다.

  - 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정

    - 이 기반 메서드들은 골격 구현에서는 추상 메서드가 될 것이다.

  - 그 다음 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공

    - 단, `equals`와 `hashCode` 같은 `Object`의 메서드는 디폴트 메서드로 제공하면 안된다.

  - 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 만들어 남은 메서드들을 작성해 넣는다.

    - 인터페이스의 메서드 모두가 기반 메서드와 디폴트 메서드가 된다면 골격 구현 클래스를 별도로 만들 이유는 없다.

  - ```java
    public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {
    ```

    - 기반 메서드

      - `K getKey();`
      - `V getValue();`

    - 디폴트 메서드

      - ```java
        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
          return (Comparator<Map.Entry<K, V>> & Serializable)(c1, c2) -> 
            c1.getKey().compareTo(c2.getKey());
        }
        ```

        - 기반 메서드를 쓰고 있고있다.
          - comparingByKey()가 호출하는 getKey() 메서드는 골격 구현 클래스를 확장하는 클래스가 재정의하는 getKey() 메서드  

    - 골격 구현 클래스 메서드

      - 인터페이스, 추상 클래스의 추상 메서드는 구현이 강제 된다.
        - 인터페이스를 구현한 추상 클래스는 인터페이스의 메서드의 구현이 강제 되지 않는다.
        - 추상 클래스를 확장하는 클래스에서 인터페이스와 추상클래스의 메서드들의 구현이 강제 되겠지.

```java
public interface A {

    void test1A();

    default void test2A() {
        System.out.println("test2A()");
    }

}

public abstract class B implements A {

    abstract void test1B();

    void test2B() {
        System.out.println("test2B()");
    }

}

public class C implements A {

    @Override
    public void test1A() {}

}

public class D extends B {

    @Override
    public void test1A() {}

    @Override
    void test1B() {}

}
```



- 골격 구현은 기본적으로 상속해서 사용하는 걸 가정하므로 아이템 19에서 이야기한 설계 문서화 지침을 모두 따라야 한다.
  - 인터페이스에 정의한 디폴트 메서드든 별도의 추상 클래스든, 골격 구현은 반드시 그 동작 방식을 잘 정리해 문서로 남겨야 한다.
- 단순 구현(simple implementation)은 골격 구현의 작은 변종으로, `AbstractMap.SimpleEntry`가 좋은 예다.
  - 단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니란 점이 다르다.
  - 동작하는 가장 단순한 구현인 단순 구현은 그대로 써도 되고 필요에 맞게 확장해도 된다.



## 아이템 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

- 자바 8 전에는 기존 구현체를 깨뜨리지 않고는 인터페이스에 메서드를 추가할 방법이 없었다.
  - 자바 8에 기존 인터페이스에 메서드를 추가할 수 있도록 디폴트 메서드가 나왔다.
    - 그렇다고 위험이 완전히 사라진 것은 아니다.
    - 인터페이스에 새로운 기능이 생길거라고 생각도 안하고 있던 인터페이스 구현체들에 디폴트 구현이 들어가기 때문이다. 



- 자바 8에는 핵심 컬렉션 인터페이스에 다수의 디폴트 메서드가 추가 되었다.
  - 주로 람다를 활용하기 위해서
  - 코드 품질이 높고 범용적이라 대부분 상황에 잘 작동한다.
  - 하지만 생각할 수 있는 모든 상황에 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운 법



- org.apache.commons.collections4.collection.**SynchronizedCollection**의 예
  - 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스(아이템 18)다.
  - 이 책이 나온 시점에는 자바 8의 Collection 인터페이스에 추가된 removeIf 메서드를 재정의 하지 않았다고 한다.
  - `ConcurrentModificationException`이 발생하거나 예기치 못한 결과로 이어질 수 있다.



- 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다.
  - 추가하려는 디폴트 메서드가 기존 구현체들과 충돌하지는 않을지 심사숙고해야 한다.



- 새로운 인터페이스를 만드는 경우라면 표준적인 메서드 구현을 제공하는 데 아주 유용한 수단이며, 그 인터페이스를 더 쉽게 구현해 활용할 수 있게끔 해준다(아이템 20).



- 디폴트 메서드라는 도구가 생겼더라도 인터페이스를 설계 할 때는 여전히 세심한 주의를 기울이자.
  - 디폴트 메서드로 기존 인터페이스에 새로운 메서드를 추가하면 커다란 위험도 딸려 온다.



## 아이템 22. 인터페이스는 정의하는 용도로만 사용하라

> In summary, interfaces should be used only to define types. They should not be used merely to export constants.

> 인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.



- 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것이다.
  - 인터페이스는 오직 이 용도로만 사용해야 한다.
  - 이 지침에 맞지 않는 예로 소위 상수 인터페이스라는 것이 있다.
    - 상수 인터페이스란 메서드 없이, 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스를 말한다.
    - `java.io.ObjectStreamConstants` 등 자바에는 상수 인터페이스가 몇 개 있으나 잘못 활용한 예이니 따라 하지 말자.

```java
// 코드 22-1 상수 인터페이스 안티패턴 - 사용금지!
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```



- 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 한다.
  - 모든 숫자 기본 타입의 박싱 클래스가 대표적으로, Integer와 Double에 선언된 MIN_VALUE와 MAX_VALUE 상수가 이런 예다.
- 열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개하면 된다(아이템 34).
- 이도 저도 아니면 인스턴스화할 수 없는 유틸리티 클래스(아이템 4)에 담아 공개하자.
  - 유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 함께 명시해야 한다.
    - `PhysicalConstants.AVOGADROS_NUMBER`
  - 유틸리티 클래스의 상수를 빈번히 사용한다면 정적 임포트(static import)하여 클래스 이름은 생략할 수 있다.

```java
// 코드 22-2 상수 유틸리티 클래스
public class PhysicalConstants {
    private PhysicalConstants() { }  // 인스턴스화 방지

    // 아보가드로 수 (1/몰)
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

    // 전자 질량 (kg)
    public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```



## 아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

> In summary, tagged classes are seldom appropriate. If you’re tempted to write a class with an explicit tag field, think about whether the tag could be eliminated and the class replaced by a hierarchy. When you encounter an existing class with a tag field, consider refactoring it into a hierarchy.

> 태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는 걸 고민해보자.



- 두 가지 이상의 의미를 표현할 수 있으며, 그중 현재 표현하는 의미를 태그 값으로 알려주는 클래스
  - 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가 많다.
  - 여러 구현이 한 클래스에 혼합돼 있어서 가독성도 나쁘다.
  - 다른 의미를 위한 코드도 언제나 함께 하니 메모리도 많이 사용한다.
  - 인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다.
  - 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.

```java
// 코드 23-1 태그 달린 클래스 - 클래스 계층구조보다 훨씬 나쁘다!
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```



- 자바 같은 객체 지향 언어는 타입 하나로 다양한 의미의 객체를 표현하는 훨씬 나은 수단을 제공한다.
  - 클래스 계층구조를 활용하는 서브타이핑(subtyping)
  - 태그 달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류일 뿐이다.
- 태그 달린 클래스를 클래스 계층구조로 바꾸는 방법
  - 가장 먼저 계층구조의 루트(root)가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.
  - 그런 다음 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.
  - 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다.
  - 다음으로, 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다.

```java
// 코드 23-2 태그 달린 클래스를 클래스 계층구조로 변환
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}

// 클래스 계층구조에서라면 다음과 같이 정사각형이 사각형의 특별한 형태임을 아주 간단하게 반영할 수 있다.
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```



## 아이템 24. 맴버 클래스는 되도록 static으로 만들라.

> To recap, there are four different kinds of nested classes, and each has its place. If a nested class needs to be visible outside of a single method or is too long to fit comfortably inside a method, use a member class. If each instance of a mem- ber class needs a reference to its enclosing instance, make it nonstatic; otherwise, make it static. Assuming the class belongs inside a method, if you need to create instances from only one location and there is a preexisting type that characterizes the class, make it an anonymous class; otherwise, make it a local class.

> 중첩 클래스에는 네 가지가 있으며, 각각의 쓰임이 다르다. 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만든다. 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로, 그렇지 않으면 정적으로 만들자. 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들고, 그렇지 않으면 지역 클래스로 만들자.



- 중첩 클래스(nested class)란 다른 클래스 안에 정의된 클래스를 말한다.
  - 중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.
  - 중첩 클래스의 종류
    - 정적 멤버 클래스, (비정적) 멤버 클래스, 익명 클래스, 지역 클래스




```java
// 정적 멤버 클래스
public class InnerExam1{
    static class Cal{
      int value = 0;
      public void plus(){
        value++;
      }
    }

    public static void main(String args[]){
      InnerExam1.Cal cal = new InnerExam1.Cal();
      cal.plus();
      System.out.println(cal.value);

    }
}
```

- 정적 멤버 클래스
  - 정정 멤버 클래스는 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하고는 일반 클래스와 똑같다.
  - 정적 멤버 클래스는 다른 정적 멤버와 똑같은 접근 규칙을 적용받는다.
    - private으로 선언하면 바깥 클래스에서만 접근할 수 있다.
  - 정적 멤버 클래스는 흔히 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰인다.



```java
// 비정적 멤버 클래스
public class InnerExam2{
    class Cal{
      int value = 0;
      public void plus(){
        value++;
      }
    }

    public static void main(String args[]){
      InnerExam2 t = new InnerExam2();
      InnerExam2.Cal cal = t.new Cal();
      cal.plus();
      System.out.println(cal.value);

    }
}
```

- 비정적 멤버 클래스
  - 정적 멤버 클래스와 구문상 차이가 나는 부분은 static이 붙어 있고 없고
  - 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결
    - 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.
      - 정규화된 this란 *클래스명*.this 형태로 바깥 클래스의 이름을 명시하는 용법
    - 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없다.
  - 비정적 멤버 클래스는 어댑터를 정의할 때 자주 쓰인다.
    - 즉, 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용한다.
    - Map 인터페이스의 구현체들은 보통 (keySet, entrySet, values 메서드가 반환하는) 자신의 컬렉션 뷰를 구현할 때 비정적 멤버 클래스를 사용
    - Set과 List 같은 다른 컬렉션 인터페이스 구현들도 자신의 반복자를 구현할 때 비정적 멤버 클래스를 주로 사용

```java
// 코드 24-1 비정적 멤버 클래스의 흔한 쓰임 - 자신의 반복자 구현
public class MySet<E> extends AbstractSet<E> {
  	... // 생략
      
    @Override public Iterator<E> iterator() {
      	return new MyIterator();
    }  
  
  	private class MyIterator implements Iterator<E> {
      	...
    }
}
```



- 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.
  - static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다.
    - 이 참조를 저장하려면 시간과 공간이 소비된다.
      - 더 심각한 문제는 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다는 점이다(아이템 7).
        - 참조가 눈에 보이지 않으니 문제의 원인을 찾기 어려워 때때로 심각한 상황을 초래하기도 한다.
- 멤버 클래스가 공개된 클래스의 public이나 protected 멤버라면 정적이냐 아니냐는 두 배로 중요해진다.
  - 멤버 클래스 역시 공개 API가 되니, 혹시라도 향후 릴리스에서 static을 붙이면 하위 호환성이 깨진다.



```java
// 지역 클래스
public class InnerExam3{
    public void exec(){
      class Cal{
        int value = 0;
        public void plus(){
          value++;
        }
      }
      Cal cal = new Cal();
      cal.plus();
      System.out.println(cal.value);
    }


    public static void main(String args[]){
      InnerExam3 t = new InnerExam3();
      t.exec();
    }
}
```

- 지역 클래스는 네 가지 중첩 클래스 중 가장 드물게 사용된다.

- 익명 클래스는 이제는 람다에게 그 자리를 물려줬다(아이템 42).



## 아이템 25. 톱 레벨 클래스는 한 파일에 하나만 담아라

> The lesson is clear: **Never put multiple top-level classes or interfaces in a single source file.** Following this rule guarantees that you can’t have multiple definitions for a single class at compile time. This in turn guarantees that the class files generated by compilation, and the behavior of the resulting program, are independent of the order in which the source files are passed to the compiler.

> 교훈은 명확하다. 소스 파일 하나에는 반드시 톱레벨 클래스 (혹은 톱레벨 인터페이스)를 하나만 담자. 이 규칙만 따른다면 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어 내는 일은 사라진다. 소스 파일을 어떤 순서로 컴파일하든 바이너리 파일이나 프로그램의 동작이 달라지는 일은 결코 일어나지 않을 것이다.



- 소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 불평하지 않는다. 
  - 하지만 아무런 득이 없을 뿐더러 심각한 위험을 감수해야 하는 행위다.
  - 이렇게 하면 한 클래스를 여러 가지로 정의할 수 있으며, 그중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하냐에 따라 달라지기 때문이다.
- 톱레벨 클래스들을 서로 다른 소스 파일로 분리
  - 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스(아이템 24)를 사용하는 방법을 고민해볼 수 있다.
  - 다른 클래스에 딸린 부차적인 클래스라면 정적 멤버 클래스로 만드는 쪽이 일반적으로 더 나을 것이다.
    - 읽기 좋고 , private으로 선언하면(아이템 15) 접근 범위도 최소로 관리할 수 있기 때문이다.

```java
// 코드 25-3 톱레벨 클래스들을 정적 멤버 클래스로 바꿔본 모습
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

