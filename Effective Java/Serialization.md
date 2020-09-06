

## 아이템 85. 자바 직렬화의 대안을 찾으라

> In summary, serialization is dangerous and should be avoided. If you are designing a system from scratch, use a cross-platform structured-data representation such as JSON or protobuf instead. Do not deserialize untrusted data. If you must do so, use object deserialization filtering, but be aware that it is not guaranteed to thwart all attacks. Avoid writing serializable classes. If you must do so, exercise great caution. 

> 직렬화는 위험하니 피해야 한다. 시스템을 밑바닥부터 설계한다면 JSON이나 프로토콜 버퍼 같은 대안을 시용하자. 신뢰할 수 없는 데이터는 역직렬화하지 말자. 꼭 해야 한다면 객체 역직렬화 필터링을 사용하되, 이마저도 모든 공격을 막아줄 수는 없음을 기억하자. 클래스가 직렬화를 지원하도록 만들지 말고, 꼭 그렇게 만들어야 한다면 정말 신경써서 작성해야 한다.



#### 직렬화 도입

- 1997년 자바에 처음으로 직렬화가 도입
  - 장점
    - 프로그래머가 어렵지 않게 분산 객체를 만들 수 있다는 구호는 매력적이었음.
  - 단점
    - 보이지 않는 생성자
    - API와 구현 사이의 모호해진 경계
    - 잠재적인 정확성 문제
    - 성능
    - 보안
    - 유지보수성



#### 보안 문제

- 직렬화의 보안 문제
  - 2000년대 초반에 논의된 취약점들이 그 후로 십 년 이상 심각하게 악용됨.
  - 2016년 11월에는 샌프란시스코 시영 교통국이 랜섬웨어 공격을 받아 요금 징수 시스템이 이틀간 마비되는 사태를 겪기도 했다.
  - 직렬화의 근본적인 문제는 공격 범위가 너무 넓고 지속적으로 더 넓어져 방어하기 어렵다는 점이다.
  - 사실상 마법 같은 생성자 `readObject` 메서드
    - `ObjectInputStream`의 `readObject` 메서드를 호출하면 객체 그래프가 역직렬화 된다.
    - `readObject` 메서드는 (`Serializable` 인터페이스를 구현했다면) 거의 모든 타입의 객체를 만들어 낼 수 있다.
    - 바이트 스트림을 역직렬화하는 과정에서 `readObject` 메서드는 `Serializable` 인터페이스를 구현한 타입들 안의 모든 코드를 수행할 수 있다.
      - `Serializable` 인터페이스를 구현한 타입들의 코드 전체가 공격 범위에 들어간다는 뜻임.



#### 가젯 체인

- 가젯(gadget)
  - 역직렬화 과정에서 호출되서 잠재적으로 위험한 동작을 수행하는 메서드
  - 자바 라이브러리, 널리 쓰이는 서드파티 라이브러리 등에서 직렬화 가능 타입들을 연구하여 찾음.
  - 가젯 체인
    - 여러 가젯을 함께 사용하여 가젯 체인을 구성할 수도 있다.
    - 가끔씩 공격자가 기반 하드웨어의 네이티브 코드를 마음대로 실행할 수 있는 아주 강력한 가젯 체인도 발견 되기도 한다.
      - 그래서 아주 신중하게 제작한 바이트 스트림만 역질렬화해야 한다.
    - 샌프란시스코 교통국을 마비시킨 공격이 정확히 이런 사례로, 가젯들이 체인으로 엮여 피해가 더 컸다고



#### 역질렬활 폭탄

- 역직렬화 폭탄
  - 이 예제를 역질렬화하려면 `hashCode` 메서드를 2^100번 넘게 호출해야 한다.

```java
static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();
    for (int i = 0; i < 100; i++) {
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();
        t1.add("foo"); // t1을 t2와 다르게 만든다.

        s1.add(t1);
        s1.add(t2);

        s2.add(t1);
        s2.add(t2);

        s1 = t1;
        s2 = t2;
    }
    return serialize(root);
}

// s1에 root 주소값 넣고 거기에 t1, t2 add
// for문 돌면서 s1, s2에 각각 t1, t2 주소 넣고 s1, s2에 t1, t2 주소값 넣는거 반복
```



#### 어떻게 대처해야 할까

- **직렬화 위험을 회피하는 가장 좋은 방법은 아무젓도 역직렬화하지 않는 것이다.**
  - 애초에 신뢰할 수 없는 바이트 스트림을 역질렬화하는 일 자체가 스스로 공격에 노출하는 행위다.
- 새로운 시스템에서 자바 직렬화를 써야 할 이유는 전혀 없다.
  - 객체와 바이트 시퀀스를 변환해주는 다른 메커니즘이 많이 있다.
    - 크로스-플랫폼 구조화 된 데이터 표현(cross-platform structured-data representation)
      - 자바 직렬화의 여러 위험을 회피
      - 다양한 플랫폼 지원
      - 우수한 성능
      - 풍부한 지원 도구
      - 활발한 커뮤니티와 전문가 집단



#### 크로스-플랫폼 구조화된 데이터 표현

- 이 표현들의 공통점은 자바 직렬화보다 훨씬 간단하다는 점.
- 임의의 객체 그래프를 자동으로 직렬화/역직렬화 하지 않는다.
- 속성-값 쌍의 집합으로 구성된 간단하고 구조화된 데이터 객체를 사용
  - 기본 타입 몇 개와 배열 타입만 지원한다.
  - 이런 간단한 추상화만으로도 아주 강력한 분산 시스템을 구축하기 충분함.
  - 자바 직렬화가 가져온 심각한 문제들을 회피할 수 있음이 밝혀졌다.

- 대표 주자
  - JSON
    - 언어 중립적이라고 하지만, JSON은 자바스크립트용으로 아직도 그 흔적이 남아있다.
    - 텍스트 기반이라 사람이 읽을 있어 텍스트 기반 표현에는 아주 효과적이다.
  - 프로토콜 버퍼(Protocol Buffers 혹은 짧게 protobuf)
    - 구글이 서버 사이에 데이터를 교환하고 저장하기 위해 설계함.
    - 언어 중립적이라고 하지만, 프로토콜 버퍼는 C++용으로 아직도 그 흔적이 남아있다.
    - 이진 표현이라 효율이 훨씬 높다.
    - 문서를 위한 스키마(타입)를 제공하고 올바로 쓰도록 강요하고 있다.
    - 이진 표현 뿐만 아니라 사람이 읽을 수 있는 텍스트 표현(pbtxt)도 지원한다.



#### 레거시 시스템

- 레거시 시스템 때문에 자바 직렬화를 완전히 배제할 수 없을 때의 차선책은? 
  - 신뢰할 수 없는 데이터의 역직렬화는 본질적으로 위험하므로 절대로 피해야 한다.
  - 특히, 신뢰할 수 없는 발신원으로부터의 RMI는 절대 수용해서는 안된다.
- 객체 역직렬화 필터링(`java.io.ObjectInputFilter`)
  - 직렬화를 피할 수 없고 역직렬화한 데이터가 안전한지 완전히 확신할 수 없다면 사용하자.
  - 자바 9에 추가 되었고, 이전 버전에서도 쓸 수 있도록 이식되었다.
  - 데이터 스트림이 역직렬화 되기 전에 필터를 설치하는 기능이다. 
  - 클래스 단위로, 특정 클래스를 받아들이거나 거부할 수 있다.
  - 필터링 기능은 메모리를 과하게 사용하거나 객체 그래프가 너무 깊어지는 사태로부터도 보호 해준다.
    - 하지만 앞의 예제의 직렬화 폭탄은 못 걸러낸다고 한다.
  - 두가지 방식
    - 블랙리스트 방식
      - '기본 수용' 모드에서는 블랙리스트에 기록된 잠재적으로 위험한 클래스들을 거부
    - 화이트리스트 방식
      - '기본 거부' 모드에서는 화이트리스트에 기록된 안전하다고 알려진 클래스들만 수용
      - 블랙리스트 방식은 이미 알려진 위험으로부터만 보호할 수 있기 때문에 화이트리스트 방식을 추천
      - 우리가 만드는 애플리케이션의 화이트리스트를 자동으로 생성해주는 스왓(SWAT, Serial Whitelist Application Trainer)



#### 기타

- 참고
  - [Java Object Deserialization을 이용한 원격코드 실행](http://blog.naver.com/PostView.nhn?blogId=skinfosec2000&logNo=220563145530&parentCategoryNo=&categoryNo=11&viewDate=&isShowPopularPosts=true&from=search)



## 아이템 86. Serializable을 구현할지는 신중히 결정하라

> To summarize, the ease of implementing Serializable is specious. Unless a class is to be used only in a protected environment where versions will never have to interoperate and servers will never be exposed to untrusted data, implementing Serializable is a serious commitment that should be made with great care. Extra caution is warranted if a class permits inheritance. 

> Serializable은 구현한다고 선언하기는 아주 쉽지만, 그것은 눈속임일 뿐이다. 한 클래스의 여러 버전이 상호작용할 일이 없고 서버가 신뢰할 수 없는 데이터에 노출될 가능성이 없는 등, 보호된 환경에서만 쓰일 클래스가 아니라면 Serializable 구현은 아주 신중하게 이뤄져야 한다. 상속할 수 있는 클래스라면 주의사항이 더욱 많아진다.



- 어떤 클래스의 인스턴스를 직렬화하고 싶으면 클래스 선언 뒤에 `implements Serializable`만 덧붙이면 된다.
  - 너무 쉽게 적용 가능해서 특별히 신경 쓸게 없다고 생각할 수 있다.
    - 하지만 실제로는 아주 복잡하다.
    - 직렬화를 지원하는건 길게 보면 아주 값비싼 일이다. 



#### 어려워진 수정

- **`Serializable`을 구현하면 릴리스한 뒤에는 수정하기 어렵다.**
  - 클래스가 `Serializable`을 구현하면 클래스의 직렬화된 바이트 스트림도 공개 API가 된다.
    - (다른 공개 API와 마찬가지로) 영원히 지원해야 한다.
  - 커스텀 직렬화 형태(아이템 87)를 설계하지 않고 자바의 기본 방식을 사용하면 캡슐화가 깨진다.
    - 기본 직렬화 형태에서는 클래스의 `private`과 `package-private` 인스턴스 필드들 마저 API로 공개된다.
    - 필드로의 접근을 최대한 막아 정보를 은닉하라는 조언(아이템 15)도 무력화된다.
  - 직렬화 가능 클래스를 만들고자 한다면, 길게 보고 감당할 수 있을 만큼 고품질의 직렬화 형태도 주의해서 함께 설계해야 한다(아이템 87, 90)
  - 직렬화가 클래스 개선을 방해하는 간단한 예 
    - 스트림 고유 식별자, 즉 직렬 버전 UID(serial version UID)
    - serialVersionUID라는 이름의 `static final long` 필드에 값을 정해주지 않으면 런타임에 값이 해시 함수를 사용해서 자동으로 설정된다.
      - 이 값을 생성하는 데는 클래스 이름, 구현한 인터페이스 등 대부분의 클래스 멤버들이 고려된다.
    - 나중에 편의 메서드를 추가하는 등 뭔가 하나라도 수정을 하면 직렬 버전 UID 값도 변한다.
      - 자동 생성되는 값에 의존하면 쉽게 호환성이 깨지고 런타임에 `InvalidClassException`이 발생한다.



#### 높아진 보안 위험

- **`Serializable`을 구현하면 버그와 보안 구멍이 생길 위험이 높아진다(아이템 85).**
  - 객체는 생성자를 사용해 만드는 게 기본이다.
  - 직렬화는 언어의 기본 메커니즘을 우회하는 객체 생성 기법이다.
  - 기본 역질렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다(아이템 88).



#### 늘어난 테스트

- **`Serializable`을 구현하면 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다.**
  - 직렬화 가능 클래스가 수정되면 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화할 수 있는지, 그리고 그 반대도 가능한지를 검사해야 한다.



#### Serializable 구현 여부

- **`Serializable` 구현 여부는 가볍게 결정할 사안이 아니다.**
  - 객체를 전송하거나 저장할 때 자바 직렬화를 이용하는 프레임워크용으로 만든 클래스라면 선택의 여지가 없다.
  - `Serializable`을 반드시 구현해야 하는 다른 클래스의 컴포넌트로 쓰일 클래스도 마찬가지다.

- `Serializable` 구현에 따르는 비용이 적지 않으니, 클래스를 설계할 때마다 그 이득과 비용을 잘 저울질해야 한다.
  - `BigInteger`와 `Instant` 같은 '값' 클래스와 컬렉션 클래스들은 `Serializable`을 구현
  - 객체를 표현하는 클래스들은 대부분 `Serializable`을 구현하지 않았다.



#### 상속용으로 설계된 클래스

- **상속용으로 설계된 클래스(아이템 19)는 대부분 `Serializable`을 구현하면 안 되며, 인터페이스도 대부분 `Serializable`을 확장해서는 안 된다.**
  - 이 규칙을 어겨야 하는 상황도 있다.
    - `Serializable`을 구현한 클래스만 지원하는 프레임워크를 사용하는 상황이라면 다른 방도가 없다.
- 상속용으로 설계된 클래스 중 `Serializable`을 구현한 예로는 `Throwable`과 `Component`가 있다.



#### 주의할 점

- 작성하는 클래스의 인스턴스 필드가 직렬화와 확장이 모두 가능하다면 주의할 점

  - 인스턴스 필드 값 중 불변식을 보장해야 할 게 있다면 반드시 하위 클래스에서 `finalize` 메서드를 재정의하지 못하게 해야 한다.

    - The `finalize()` method is called the `finalizer`.

      - `Object`에 있다.

        - ```java
          @Deprecated(since="9")
          protected void finalize() throws Throwable { }
          ```

    - `finalize` 메서드를 자신이 재정의하면서 `final`로 선언하면 된다.

      - 이렇게 해두지 않으면 `finalizer` 공격(아이템 8)을 당할 수 있다.

  - 인스턴스 필드 중 기본값(정수형은 0, `boolean`은 `false`, 객체 참조 타임은 `null`)으로 초기화되면 위배되는 불변식이 있다면 클래스에 다음의 readObjectNoData 메서드를 반드시 추가해야 한다.

    - 기본값으로 초기화되면

      - 따로 초기화를 안해주면 디폴트로 들어오는 값.

    - 자바 4에서 추가된 메서드

    - 기존의 직렬화 가능 클래스에 직렬화 가능 상위 클래스를 추가하는 드문 경우를 위한 메서드

      - 볼일이 있을라나?

    - ```java
      // 코드 86-1 상태가 있고, 확장 가능하고, 직렬화 가능한 클래스용 readObjectNoData 메서드
      private void readObjectNoData() throws InvalidObjectException {
          throw new InvalidObjectException("스트림 데이터가 필요합니다");
      }
      ```

- `Serializable`을 구현하지 않기로 할 때는 한 가지만 주의하면 된다.

  - 상속용 클래스인데 직렬화를 지원하지 않으면 그 하위 클래스에서 직렬화를 지원하려 할 때 부담이 늘어난다.
    - 이런 클래스를 역질렬화하려면 그 상위 클래스가 매개변수가 없는 생성자를 제공해줘야 한다.
      - 이런 생성자를 제공하지 않으면 하위 클래스에서는 어쩔 수 없이 직렬화 프록시 패턴(아이템 90)을 사용해야 한다.



#### 내부 클래스

- **내부 클래스(아이템 24)는 직렬화를 구현하지 말아야 한다.**
  - 내부 클래스에는 컴파일러가 생성하는 필드들이 자동으로 추가된다.
    - 바깥 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해
  - 내부 클래스에 대한 기본 직렬화 형태는 분명하지가 않다.
    - 단, 정적 멤버 클래스는 `Serializable`을 구현해도 된다.



## 아이템 87. 커스텀 직렬화 형태를 고려해보라

> To summarize, if you have decided that a class should be serializable (Item 86), think hard about what the serialized form should be. Use the default serialized form *only* if it is a reasonable description of the logical state of the object; otherwise design a custom serialized form that aptly describes the object. You should allocate as much time to designing the serialized form of a class as you allocate to designing an exported method (Item 51). Just as you can’t eliminate exported methods from future versions, you can’t eliminate fields from the serialized form; they must be preserved forever to ensure serialization compatibility. Choosing the wrong serialized form can have a permanent, negative impact on the complexity and performance of a class.


> 클래스를 직렬화하기로 했다면(아이템 86) 어떤 직렬화 형태를 사용할지 심사숙고하기 바란다. 자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때만 사용하고, 그렇지 않으면 객체를 적절히 설명하는 커스텀 직렬화 형태를 고안하라. 직렬화 형태도 공개 메서드(아이템 51)를 설계할 때에 준하는 시간을 들여 설계해야 한다. 한번 공개된 메서드는 향후 릴리스에서 제거할 수 없듯이, 직렬화 형태에 포함된 필드도 마음대로 제거할 수 없다. 직렬화 호환성을 유지하기 위해 영원히 지원해야 하는 것이다. 잘못된 직렬화 형태를 선택하면 해당 클래스의 복잡성과 성능에 영구히 부정적인 영향을 남긴다.



#### 기본 직렬화

- **먼저 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라.**
  - 기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용해야 한다.
  - 일반적으로 직접 설계했는데 기본 직렬화 형태와 거의 같은 결과가 나올 경우에만 기본 형태를 써야 한다.
- 어떤 객체의 기본 직렬화 형태는 객체가 포함한 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내며, 심지어 이 객체들이 연결된 위상(topology)까지 기술한다.
  - 그러나 아쉽게도 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다.
- **객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.**
  - 코드 87-1 예제의 성명은 논리적으로 이름, 성, 중간이름이라는 3개의 문자열로 구성되며, 앞 코드인 인스턴스 필드들은 이 논리적 구성요소를 정확히 반영했다.

- **기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 `readObject` 메서드를 제공해야 할 때가 많다.**
  - 아이템 88과90에서 길게 다룬다.

```java
// 코드 87-1 기본 직렬화 형태에 적합한 후보
public class Name impements Serializable {
	 /**
    * 성. null이 아니어야 함.
    * @serial
    */
    private final String lastName;
 
	 /**
    * 이름. null이 아니어야 함.
    * @serial
    */
    private final String firstName;
 
   /**
    * 중간이름. 중간이름이 없다면 null.
    * @serial
    */
    private final String middleName;
 
		... // 나머지 코드는 생략
}
```



- **객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 크게 네 가지 면에서 문제가 생긴다.**
  - **공개 API가 현재의 내부 표현 방식에 영구히 묶인다.**
    - 코드 87-2 예제의 StringList.Entry가 공개 API가 된다.
      - 다음 릴리스에서 내부 표현 방식을 바꾸더라도 StringList 클래스는 연결 리스트를 처리할 수 있어야 한다.
  - **너무 많은 공간을 차지할 수 있다.**
    - 포함될 필요가 없는 엔트리와 연결 정보 같은 내부 구현 등이 저장된다.
      - 직렬화 형태가 너무 커지면 디스크에 저장하고나 네트워크로 전송하는 속도가 느려진다.
  - **시간이 너무 많이 걸릴 수 있다.**
  - **스택 오버플로를 일으킬 수 있다.**
    - 기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 이 작업은 중간 정도 크기의 객체 그래프에서도 자칫 스택 오버플로를 일으킬 수 있다.

```java
// 코드 87-2 기본 직렬화 형태에 적합하지 않은 클래스
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;
 
    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    ... // 나머지 코드는 생략
}
```



#### 커스텀 직렬화

- 코드 87-3 예제에서 `writeObject`와 `readObject`가 직렬화 형태를 처리한다.
  - 일시적이란 뜻의 `transient` 한정자는 해당 인스턴스 필드가 기본 직렬화 형태에 포함되지 않는다는 표시다.

- `writeObject`와 `readObject`는 각각 `defaultWriteObject`와 `defaultReadObject`를 호출하고 있다.
  - StringList의 필드에 모두 `transient`가 붙어서 기본 직렬화 형태에 포함되지 않는데도 말이다.
    - 직렬화 명세는 그럼에도 불구하고 이 작업을 무조건 하라고 요구하고 있다.
  - 이렇게 하면 향후 릴리스에서 `transient`가 아닌 인스턴스 필드가 추가되더라도 상호(상위와 하위 모두) 호환되기 때문이다.
    - 신버전 인스턴스를 직렬화한 후 구버전으로 역질격화하면 새로 추가된 필드들은 무시될 것이다.
- 불변식이 세부 구현에 따라 달라지는 객체는 정확하게 복원이 안될 수 있다.
  - 계산할 때마다도 달라지기도 하는 해시테이블 같은 경우는 직렬화한 후 역질렬화하면 불변식이 심각하게 훼손된 객체들이 생겨날 수 있다. 

```java
// 코드 87-3 합리적인 커스텀 직렬화 형태를 갖춘 StringList
public final class StringList implements Serializable {
    private transient int size   = 0;
    private transient Entry head = null;

    // 이제는 직렬화되지 않는다.
    private static class Entry {
        String data;
        Entry  next;
        Entry  previous;
    }

    // 지정한 문자열을 이 리스트에 추가한다.
    public final void add(String s) {  }

    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     *
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
     * 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s)
            throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);

        // 모든 원소를 올바른 순서로 기록한다.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }

    private void readObject(ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();

        // 모든 원소를 읽어 이 리스트에 삽입한다.
        for (int i = 0; i < numElements; i++)
            add((String) s.readObject());
    }

    // 나머지 코드는 생략
}
```



#### transient

- 기본 직렬화를 수용하든 하지 않든 `defaultWriteObject` 메서드를 호출하`면 transient`로 선언하지 않은 모든 인스턴스 필드가 직렬화된다.
- **해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 `transient` 한정자를 생략해야 한다.**
  - 커스텀 직렬화 형태를 사용한다면, 앞서의 StringList 예에서처럼 대부분의 (혹은 모든) 인스턴스 필드를 `transient`로 선언해야 한다.
- 기본 직렬화를 사용하면 `transient` 필드들은 역질렬화될 때 기본값으로 초기화됨을 잊지 말자.
  - 객체 참조 필드는 `null`로, 숫자 기본 타입 필드는 0으로, `boolean` 필드는 `false`로 초기화된다.
- 기본 값을 그대로 사용해서는 안 된다면 `readObject` 메서드에서 `defaultReadObject`를 호출한 다음, 해당 필드를 원하는 값으로 복원하자(아이템 88).
  - 혹은 그 값을 처음 사용할 때 초기화하는 방법도 있다(아이템 83).



#### 동기화

- 기본 직렬화 사용 여부와 상관없이 **객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다.**
  - 모든 메서드를 `synchronized`로 선언하여 스레드 안전하게 만든 객체(아이템 82)
    - 이 객체에서 기본 직렬화를 사용하려면 `writeObject`도 코드 87-4 처럼 `synchronized`로 선언해야 한다.
      - `writeObject` 안에서 동기화하고 싶다면 클래스의 다른 부분에서 사용하는 락 순서를 똑같이 따라야 한다.
        - 그렇지 않으면 자원 순서 교착상태(resource-ordering deadlock)에 빠질 수 있다.

```java
// 코드 87-4 기본 직렬화를 사용하는 동기화된 클래스를 위한 writeObject 메서드
private synchronized void writeObject(ObjectOutputStream s) throws IOException{
    s.defaultWriteObject();
}
```



#### 직렬 버전 UID

- **어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여하자.**
  - 이렇게 하면 직렬 버전 UID가 일으키는 잠재적인 호환성 문제가 사라진다(아이템 86).
  - 성능도 빨라진다.
    - 직렬 버전 UID를 명시하지 않으면 런타임에 이 값을 생성하느라 복잡한 연산을 수행해야 한다.
  - 직렬 버전 UID 선언은 각 클래스에 한 줄만 추가해주면 된다.
    - `private static final long serialVersionUID = <무작위로 고른 long 값>;`
- 기존 버전 클래스와의 호환성을 끊고 싶다면 단순히 직렬 버전 UID의 값을 바꿔주면 된다.
- 구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정하지 말자.




## 아이템 88. readObject 메서드는 방어적으로 작성하라

> To summarize, anytime you write a readObject method, adopt the mind-set that you are writing a public constructor that must produce a valid instance regard- less of what byte stream it is given. Do not assume that the byte stream represents an actual serialized instance. While the examples in this item concern a class that uses the default serialized form, all of the issues that were raised apply equally to classes with custom serialized forms. Here, in summary form, are the guidelines for writing a readObject method:
>
> - For classes with object reference fields that must remain private, defensively copy each object in such a field. Mutable components of immutable classes fall into this category.
>
> - CheckanyinvariantsandthrowanInvalidObjectExceptionifacheckfails. The checks should follow any defensive copying.
>
> - If an entire object graph must be validated after it is deserialized, use the ObjectInputValidation interface (not discussed in this book).
>
>   • Do not invoke any overridable methods in the class, directly or indirectly.

> readObject 메서드를 작성할 때는 언제나 public 생성자를 작성하는 자세로 임해야 한다. readObject는 어떤 바이트 스트림이 넘어오더라도 유효한 인스턴스를 만들어내야 한다. 바이트 스트림이 진짜 직렬화된 인스턴스라고 가정해서는 안 된다. 이번 아이템에서는 기본 직렬화 형태를 사용한 클래스를 예로 들었지만 커스텀 직렬화를 사용하더라도 모든 문제가 그대로 발생할 수 있다. 이어서 안전한 readObject 메서드를 작성하는 지침을 요약해보았다.
>
> - private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라. 불변 클래스 내의 가변 요소가 여기 속한다.
> - 모든 불변식을 검사하여 어긋나는 게 발견되는 InvalidObjectException을 던진다. 방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다.
> - 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라(이 책에서는 다루지 않는다).
> - 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자.





```java
// 코드 88-1 방어적 복사를 사용하는 불변 클래스
public final class Period {
    private final Date start;
    private final Date end;
  
    /**
     * @param  start 시작 시각
     * @param  end 종료 시각. 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(this.start + "가 " + this.end + "보다 늦다.");
    }
  
    public Date start() {
        return new Date(start.getTime());
    }
  
    public Date end() {
        return new Date(end.getTime());
    }
  
    @Override
    public String toString() {
        return start + " - " + end;
    }
}
```







```java
// 코드 88-2 허용되지 않는 Period 인스턴스를 생성할 수 있다
public class BogusPeriod {
    // 진짜 Period 인스턴스에서는 만들어질 수 없는 바이트 스트림
    private static final byte[] serializedForm = {
            (byte) 0xac, (byte) 0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
            0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte) 0xf8,
            0x2b, 0x4f, 0x46, (byte) 0xc0, (byte) 0xf4, 0x02, 0x00, 0x02,
            0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
            0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
            0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
            0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
            0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
            0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
            (byte) 0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
            0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte) 0xdf,
            0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
            0x77, 0x08, 0x00, 0x00, 0x00, (byte) 0xd5, 0x17, 0x69, 0x22,
            0x00, 0x78};
  
    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }
  
  	// 주어진 직렬화 형태(바이트 스트림)로부터 객체를 만들어 반환한다.
    private static Object deserialize(byte[] sf) {
        try {
          	return new ObjectInputStream(
              	new ByteArrayInputStream(sf)).readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```







```java
// 코드 88-3 유효성 검사를 수행하는 readObject 메서드 - 아직 불완전
public final class Period implements Serializable {
    // 코드 생략
  
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
  
        // 불변식을 만족하는지 검사
        if (start.compareTo(end) > 0) {
            throw new InvalidObjectException(this.start + "가 " + this.end + "보다 늦다.");
        }
    }
}
```









```java
// 코드 88-4 가변 공격의 예
public class MutablePeriod {
  
    private final Period period;
    private final Date start;
    private final Date end;
  
    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bos);
  
            // 유효한 Period 인스턴스를 직렬화한다.
            out.writeObject(new Period(new Date(), new Date()));
  
            /*
             * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
             * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고하자.
             */
            byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 참조 #5
            bos.write(ref); // 시작(start) 필드
            ref[4] = 4; // 참조 #4
            bos.write(ref); // 종료(end) 필드
  
            // Period 역직렬화 후 Date 참조를 '훔친다'.
            ObjectInputStream in = new ObjectInputStream(
              											new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }
  
    public static void main(String[] args) {
        MutablePeriod mp = new MutablePeriod();
        Period p = mp.period;
        Date pEnd = mp.end;
  
        pEnd.setYear(50);
        System.out.println(p);
  
        Date pStart = mp.start;
        pStart.setYear(122);
        System.out.println(p);
    }
}
```





```java
// 코드 88-5 방어적 복사와 유효성 검사를 수행하는 readObject 메서드
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 불변식을 만족하는지 검사한다.
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(this.start + "가 " + this.end + "보다 늦다.");
    }
}
```







## 아이템 89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라


> To summarize, use enum types to enforce instance control invariants wherever possible. If this is not possible and you need a class to be both serializable and instance-controlled, you must provide a readResolve method and ensure that all of the class’s instance fields are either primitive or transient.

> 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 한 열거 타입을 사용하자. 여의치 않은 상황에서 직렬화와 인스턴스 통제가 모두 필요하다면 readResolve 메서드를 작성해 넣어야 하고, 그 클래스에서 모든 참조 타입 인스턴스 필드를 transient로 선언 해야 한다.



```java
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
   
    public void leaveTheBuilding() { ... }
}
```

```java
// 인스턴스 통제를 위한 readResolve -개선의 여지가 있다!
private Object readResolve() {
  	// 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
}
```



```java
// 코드 89-1 잘못된 싱글턴 - transient가 아닌 참조 필드를 가지고 있다!
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};
  
    private Elvis() {
    }
  
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
  
    private Object readResolve() {
        return INSTANCE;
    }
}
```







```java
// 코드 89-2 도둑 클래스
public class ElvisStealer implements Serializable {
    private static final long serialVersionUID = 0;
    static Elvis impersonator;
    private Elvis payload;
  
    private Object readResolve() {
        // resolve되기 전의 Elvis 인스턴스의 참조를 저장한다.
        impersonator = payload;
        // favoriteSongs 필드에 맞는 타입의 객체를 반환한다.
        return new String[] { "A Fool Such as I" };
    }
}
```









```java
// 코드 89-3 직렬화의 허점을 이용해 싱글턴 객체를 2개 생성한다.
public class ElvisImpersonator {
    // 진짜 Elvis 인스턴스로 만들어질 수 없는 바이트 스트림!
  	private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
         ...
    };
  
    public static void main(String[] args) {
        // ElvisStealer.impersonator를 초기화한 다음,
        // 진짜 Elvis(즉, Elvis.INSTANCE)를 반환한다.
        Elvis elvis = (Elvis) deserialize(serializedForm);
        Elvis impersonator = ElvisStealer.impersonator;
      
        elvis.printFavorites();
        impersonator.printFavorites();
    }
}
```







```java
// 코드 89-4 열거 타입 싱글턴(선호 방식) - 전통적인 싱글턴보다 우수하다.
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs =
        { "Hound Dog", "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```









## 아이템 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

> In summary, consider the serialization proxy pattern whenever you find yourself having to write a readObject or writeObject method on a class that is not extendable by its clients. This pattern is perhaps the easiest way to robustly serialize objects with nontrivial invariants.

> 제 3자가 확장할 수 없는 클래스라면 가능한 한 직렬화 프록시 패턴을 사용하자. 이 패턴이 아마도 중요한 불변식을 안정적으로 직렬화해주는 가장 쉬운 방법일 것이다.





직렬화 프록시 패턴  serialization proxy pattern



```java
// 코드 90-1 Period 클래스용 직렬화 프록시
private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;

    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }

    private static final long serialVersionUID =
            234098243823485285L; // 아무 값이나 상관없다. (아이템 87)
}
```





```java
// 직렬화 프록시 패턴용 writeReplace 메서드
private Object writeReplace() {
    return new SerializationProxy(this);
}
```





```java
// 직렬화 프록시 패턴용 readObject 메서드
private void readObject(ObjectInputStream stream)
        throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다.");
}
```





```java
private static class SerializationProxy <E extends Enum<E>> 
  			implements java.io.Serializable {
    // 이 EnumSet의 원소 타입
    private final Class<E> elementType;

  	// 이 EnumSet 안의 원소들
    private final Enum<?>[] elements;

    SerializationProxy(EnumSet<E> set) {
      	elementType = set.elementType;
      	elements = set.toArray(new Enum<?>[0]);
    }

    private Object readResolve() {
        EnumSet<E> result = EnumSet.noneOf(elementType);
        for (Enum<?> e : elements)
          	result.add((E)e);
        return result;
    }

    private static final long serialVersionUID = 362491234563181265L;
}
```

