

## 아이템 15. 클래스와 멤버의 접근 권한을 최소화하라



> To summarize, you should reduce accessibility of program elements as much as possible (within reason). After carefully designing a minimal public API, you should prevent any stray classes, interfaces, or members from becoming part of the API. With the exception of public static final fields, which serve as constants, public classes should have no public fields. Ensure that objects referenced by public static final fields are immutable.

> 프로그램 요소의 접근성은 가능한 한 최소한으로 하라. 꼭 필요한 것만 골라 최소한의 public API를 설계하자. 그 외에는 클래스, 인터페이스, 멤버가 의도치 않게 API로 공개되는 일이 없도록 해야 한다. public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져서는 안 된다. public static final 필드가 참조하는 객체가 불변인지 확인하라.



### 잘 설계된 컴포넌트

- 어설프게 설계된 컴포넌트와 잘 설계된 컴포넌트의 가장 큰 차이
  - 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼느냐.
  - 잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다.



### 정보 은닉의 장점

* 시스템 개발 속도를 높인다.
  * 여러 컴포넌트를 병렬로 개발할 수 있기 때문
  * 의존하는 모듈의 인터페이스(API)만 알면 된다.)
  
* 시스템 관리 비용을 낮춘다.
  * 각 컴포넌트가 의존관계가 얕으니 더 빨리 파악 할 수 있다.
  * 서로가 API로만 소통하니 구현체를 바꾸기 용이하다.


* 정보 은닉 자체가 성능을 높여주지는 않지만, 성능 최적화에 도움을 준다.
  * 다른 컴포넌트에 영향을 주지 않고 원하는 컴포넌트만 최적화 할 수 있다.
    * A 컴포넌트를 수정해도 B 컴포넌트가 영향을 받지 않는다.

* 소프트웨어 재사용성을 높인다.
  * 외부에 거의 의존하지 않고 독자적으로 동작할 수 있는 컴포넌트라면 낮선 환경에서도 재사용 용이

* 큰 시스템을 제작하는 난이도를 낮춰준다.
  * 전체가 완성되지 않아도 개별 컴포넌트의 동작을 검증 할 수 있다.



### 접근성을 가능한 좁히기

- **접근 제한자를 제대로 활용하는 것이 정보 은닉의 핵심**

  - 기본 원칙은 모든 클래스와 멤버의 접근성을 **가능한 한 좁혀야** 한다는 것이다.



- 톱레벨 클래스(일반 클래스)와 인터페이스에 부여 할 수 있는 접근 수준은 `package-private`, `public` 두 가지다.
  - 보통 `package-private`을 `default` 라고 불렀었다.
    - `private`인데 같은 패키지 안에서만 쓸 수 있다고 이름이 `package-private`인가 보다.
  - `public`으로 선언하면 공개 API가 되고 `default`로 선언하면 패키지 안에서만 사용할 수 있다.
  - **그냥 습관적으로 접근 수준을 `public`으로 주지말고 패키지 외부에서 쓸 이유가 없다면 `package-private`로 선언하자.** 
    - 한 클래스에서만 사용하는 클래스나 인터페이스가 있다면 그 사용하는 클래스 안에 `private static class`로 중첩시켜보자.
  - **`public` 일 필요가 없는 클래스의 접근 수준을` package-private` 톱레벨 클래스로 좁히자.** 
    - `package-private` 톱 레벨 클래스는 내부 구현에 속하기 때문에 API가 공개되고 난 뒤에도 내부 구현을 언제든지 수정할 수 있다. 
    - jar 안에 `java.util.ArrayPrefixHelpers` class의 경우 접근 수준이 `package-private`



- 클래스의 공개 API는 모든 멤버 변수 우선 `private`으로 묶고 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한하여 `private` 제한자를` package-private`으로 풀어주자. 
  - 이런 작업이 잦다면 시스템에서 컴포넌트를 더 분해해야 하는 것은 아닌지 다시 고민해보자. 
  - `Serializable`을 구현한 클래스에서는 그 필드들도 의도치 않게 공개 API가 될 수도 있다(아이템 86, 87).
  
  - `public` 클래스에서 멤버의 접근 수준을 `package-private`에서` protected`로 변경하는 순간 그 멤버에 접근할 수 있는 대상 범위가 엄청나게 넓어진다. 
    - **`public` 클래스의 `protected` 멤버는 공개 API이므로 영원히 지원돼야 한다. 그러니 `protected` 멤버의 수는 적을수록 좋다.**



- 상위 클래스의 메서드를 재정의할 때는 그 접근 수준을 상위 클래스에서보다 좁게 설정 할 수 없다. 
  - 이 제약은 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다는 규칙(**리스코프 치환 원칙**, 아이템 10)을 지키기 위해 필요하다.
  - 클래스가 인터페이스를 구현하는 건 이 규칙의 특별한 예로 볼 수 있고, 이때 클래스는 인터페이스가 정의한 모든 메서드를 `public`으로 선언해야 한다.



- 테스트를 위해서 클래스의 `private` 멤버를 `package-private`까지 풀어주는 것은 허용할 수 있지만, 그 이상은 안된다.



- `public` 클래스의 인스턴스 필드는 되도록 `public`이 아니어야 한다(아이템 16).
  - 필드가 가변 객체를 참조하거나, `final`이 아닌 인스턴스 필드를 `public`으로 선언하면 그 필드와 관련된 모든 것은 불변식을 보장할 수 없게 된다. 
    - 필드가 수정될 때 (락 획득 같은) 다른 작업을 할 수 없게 되므로 `public` 가변 필드를 갖는 클래스는 일반적으로 스레드 안전하지 않다.
      - 이 부분은 다시 체크
  - 필드가 `final`이면서 불변 객체를 참조하더라도 문제는 여전히 남는데 내부 구현을 바꾸고 싶어도 그 `public` 필드를 없애는 방식으로는 리펙터링할 수 없게 된다.
    - 불변식은 보장이 되겠지만 필드가 공개가 되어버리니 향후에 제거 할 수가 없다.
    - 이러한 문제는 정적 필드에서도 마찬가지이나 예외가 하나 있는데 꼭 필요한 경우의 상수는 `public static final` 필드로 공개해도 좋다.
      - 이런 필드는 반드시 **기본 타입**이나 **불변 객체**를 참조해야 한다(아이템 17).



- 길이가 0이 아닌 배열은 모두 변경 가능하니 주의하자. 
  - 테스트 코드를 만들어서 실습해 보니 final이지만 변경이 되는군.
  - 따라서 클래스에서 `public static final` 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다. 
  - 해결책은 두가지다.
    -  public 배열을 private으로 만들고 public 불변 리스트를 추가
    -  배열을 private으로 만들고 그 복사본을 반환하는 public 메서드를 추가하는 방법(방어적 복사).
  - 어느 반환 타입이 더 쓰기 편할지, 성능은 어느 쪽이 나을지를 고민해 정하자.

```java
// 배열을 private
private static final Thing[] PRIVATE_VALUES = { ... };
// public 불변 리스트
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUE));
```

```java
// 배열은 private
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values(){
		// 복사본을 반환
  	return PRIVATE_VALUES.clone();
}
```



### 암묵적 접근 수준 두 가지

- 패키지가 클래스들의 묶음이듯, 모듈은 패키지들의 묶음이다.

- 자바 9에서는 모듈 시스템이라는 개념이 도입되면서 두 가지 암묵적 접근 수준이 추가되었다.
  - 모듈은 자신에 속하는 패키지 중 공개(export)할 것들을 선언한다.
    - `protected` 혹은 `public` 멤버라도 해당 패키지를 공개하지 않았다면 모듈 외부에서는 접근할 수 없다.
  - 모듈 시스템을 활용하면 클래스를 외부에 공개하지 않으면서도 같은 모듈을 이루는 패키지 사이에서는 자유롭게 공유할 수 있다.
- 이 암묵적 접근 수준들은 각각 public 수준과 protected 수준과 같으나 **그 효과가 모듈 내부로 한정**되는 변종인 것이다.
- 모듈의 JAR 파일을 자신의 모듈 경로가 아닌 애플리케이션의 클래스패스(classpath)에 두면 그 모듈 안의 모든 패키지는 마치 모듈이 없는 것 처럼 행동한다.
- 새로 등장한 이 접근 수준을 적극 활용한 대표적인 예가 바로 JDK 자체다.
  - 자바 라이브러리에서 공개하지 않은 패키지들은 해당 모듈 밖에서는 절대 접근할 수 없다.



## 아이템 16. public 클래스 안에는 public 필드를 두지 말고 접근자 메서드를 사용하라



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



- 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.
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
    - 여기서 말하는 클라이언트는 `package-private` 클래스 혹은 `private` 중첩 클래스를 사용하는 쪽을 얘기하는거 같다.
      - `package-private` 클래스 혹은 `private` 중첩 클래스에 접근할 수 있다는 것은 같은 패키지 혹은 같은 클래스에 있다는 거임.
      - 공개 API와는 상관이 없겠지?



- 자바 플랫폼 라이브러리에도 `public` 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있다.
  - 대표적인 예가 `java.awt.package` 패키지의 `Point`와 `Dimension` 클래스
    - 내부를 노출한 `Dimension` 클래스의 심각한 성능 문제는 오늘날까지도 해결되지 못했다.
    - 이 클래스들을 흉내 내지 말고, 타산지석으로 삼길 바란다.



- `public` 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어 들지만, 여전히 결코 좋은 생각이 아니다.
  - API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다. 





## 아이템 17. 변경 가능성을 최소화하라

- **불변 클래스**란 간단히 말해 그 인스턴스의 내부 값을 수정할 수 없는 클래스다.
  - 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다.
  - 자바 라이브러리에도 다양한 불변 클래스가 있는데 `String`, 기본 타입의 박싱 클래스들(래퍼 클래스, `Byte`, `Integer` 등), `BigInteger`, `BigDecimal` 이 여기 속한다.
- 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다.



###  클래스를 불변으로 만드는 다섯 가지 규칙

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

- **불변 객체**는 **단순**하다.
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
    - 좋은 예로, 불변 객체는 맵의 키와 집합(Set)의 원소로 쓰기에 안성맞춤이다.



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
          - `StringBuilder`는 `final class` 이지만, 할당된 값을 변경면 새로운 객체를 만드는 방식이 아닌 기존 할당된 값을 수정하는 것으로 처리하기에 가변객체임.
            - 클래스에 `final`은 상속을 못하게 하느 거니깐



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
    - 디버깅을 `static`에 걸면 안되고 안에 걸어야한다.
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

