

## OOP



### Why Clean Code

- SW는 한번 작성되면 최소 10번 이상 읽힌다.
  - 그래서 대충 돌아만가게 작성하면 안되고 읽기 편하도록 작성해야 한다.
- 여러분들의 주 업무는 무엇인가?
  - 매번 새로운 코드를 작성하나? 
  - 남이 작성된 코드를 읽고, 수정하고, 기능을 추가하는 일을 하나?
- 기계가 이해할 수 있는 코드는 어느 바보도 작성할 수 있다.
  - 하지만 인간이 이해할 수 있는 코드는 잘 훈련된 소프트웨어 엔지니어만이 작성할 수 있다.



### Why OOP

- Procedural
  - 알아야 할 것이 적어서 초기 진입이 쉽다.
  - 모든 프로시저가 데이터를 공유
  - 변경이 어려움
- OOP
  - 데이터와 코드가 Encapsulated
  - 데이터와 그 데이터를 조작하는 코드의 변경은 외부에 영향을 안 미침
  - 외부에 노출된 인터페이스만 변경되지 않는다면
  - 프로시저를 실행하는데 필요한 만큼의 데이터만 가짐
- 단순한 시스템은 쉬운 절차지향으로
  - 대학교 과제나, 논문 만들때 쓰는건 만들고 제출하고 버리니깐 그런건 단순하다고 할 수 있을까
    - 사실상 단순한 시스템은 없다고 보는게
- 복잡하거나 요구사항 변화가 발생할 시스템은 객체지향으로
  - 데이터의 변경이 해당 객체로만 제한되고 다른 객체에 영향을 미치지 않기 때문에
  - Encapsulation
- SW가 계속 사용된다면 요구사항은 계속 바뀐다.
- 절차지향이 처음에 쉬울지 모르나 시간이 지나면 수정하기 어려운 구조가된다.



### Object / Role / Responsibility

- 객체/클래스의 이름
  - 무엇으로 정의해야
    - RequestParser
  - 어떻게 정의하지 말고
    - JsonRequestParser
      - Json이 아니고 다른걸로 변경되면 클래스명을 바꿀건가?
- 역할은 관련된 책임의 집합
- 객체는 역할을 가짐



### 객체지향 설계 과정

- 기능을 제공할 객체 후보 선별
  - 내부에서 필요한 데이터 선별
  - 정적 설계
    - 클래스 다이어그램
- 객체 간 메시지 흐름 연결
  - 동적 설계
    - 커뮤니케이션 다이어그램
    - 시퀀스 다이어그램



### Encapsulation

- 내부적으로 어떻게 구현했는지를 감춰 내부의 변경(데이터, 코드)에 client가 변경되지 않도록
  - 코드 변경에 따른 비용 최소화
  - 상태를 가진 객체가 일을하고 client는 해달라고 얘기만해라.
- 객체지향의 기본
- Strop Watch 예제
- **Tell, Don't Ask**
  - 데이터를 요청해서 변경하고 저장하라고 하지말고
  - 무슨 기능을 실행하라
  - 데이터를 잘 알고 있는 객체에게 기능을 수행하라고 하라.
  - Encapsulation이 유지되어 변경에 영향을 안 받게 됨.
  - example
    - `if (member.getExpiredDate().getTime() < System.currentTimeMillis) {`
    - `if (member.isExpired()) {`
- Law of Demeter
  - Tell, Don't Ask를 지키는지 확인하는 방법
  - Function Structure 파트에 다시 자세히 나온다.
- Command vs Query
  - Command(Tell)
  - Query(Ask)
  - 해당 객체의 상태에 기반한 결정은 반드시 객체 내에서 이뤄져야 한다.
  - Tell, Don't Ask를 잘 지키게 해 줌.
  - Function Structure 파트에 다시 자세히 나온다.



### Polymorphism

- 한 객체가 여러가지(poly) 모습/타입(morph)을 가질 수 있다.
- 상속을 통해 다형성을 구현
  - 구현 상속
    - 수퍼 타입의 구현을 재사용
  - 인터페이스 상속
    - 타입 정의만 상속
- **상속은 객체에게 다형성을 제공**
- **재사용**
  - 고수준의 로직에서 사용하는 **저수준의 로직이 바뀌어도 고수준의 로직은 영향을 받지 않는**것
    - 인터페이스만 보고 프로그래밍하면 저수준의 로직이 아무리 바뀌어도 고수준의 로직을 재사용 할 수 있다.
  - 상속 구조를 만들어 상위 객체에 공통된 로직을 올리는건 OOP에서 얘기하는 재사용의 핵심이 아니다.



### Abstraction

- 데이터/프로세스 등을 의미가 비슷한 개념/표현으로 정의하는 과정
- 상세한 구현으로부터 개념을 도출하는 과정
- 타입 추상화
  - 공통된 데이터나 프로세스를 제공하는 객체들을 하나의 타입(인터페이스)으로 추상화하는 것
- **Programming to interface**
  - Client Code가 항상 Interface에 대한 레퍼런스를 사용해야 한다는 의미
  - Client는 구현 변경에 대해서 영향을 받지 않는다.
  - Interface Signature가 사용 가능한 모든 행위를 보여줌
  - 추상화를 통해 유연함을 얻기 위한 규칙
  - 사용되는 이유
    - 런타임에 프로그램의 행위를 변경하기 위해
      - Client Code가 interface만 알고 있다면 Client Code에 변경을 하지 않고도 **런타임에 프로그램의 행위를 변경**할 수 있다.
      - DI하는 부분은 변경이 일어나도 Client Code에는 변경이 일어나지 않는다.
        - 로그를 사용할 때 SLF4J만 쳐다보고 프로그래밍을 하면 구현체가 향후에 바뀌어도 내 코드는 수정할 필요가 없는 것처럼.
    - 유지보수 측면에서 보다 나은 프로그램을 작성할 수 있게 함
- 추상화와 개발자의 습성
  - 개발자들은 습성상 상세한 구현에 빠지다보면 상위 수준의 설계를 높치기 쉽다.
  - 추상화를 통해서 상위 수준에서의 설계를 하는데 도움을 얻을 수 있다.
- 추상화/다형성의 혜택은 구현 교체의 유현함
  - Abstract Factory
    - 서비스 로케이터 패턴
      - 프로그램 개발 환경이나 사용하느 프레임워크의 제약으로 인해 DI 패턴을 적용할 수 없는 경우 사용할 수 있다.
  - DI
    - 스프링 컨테이너 같은 놈이 디펜던시를 주입해 준다.
    - 테스트가 용이해 진다.
- 고수준과 저수준 관점에서 기능을 분리하고 설계하는 연습
  - 상품 상세 정보와 추천 상품 목록 제공 기능
    - 상품 번호를 이용해서 상품 DB에서 상세 정보를 구함.
    - Daara API를 이용해서 추천 상품 5개 구함.
    - 추천 상품이 5개 미만이면 같은 분류에 속한 상품 중 최근 한 달 판매가 많은 상품을 ERP에서 구해서 5개를 채움.
  - 고수준
    - 상품 번호로 상품 상세 정보 구함.
    - 추천 상품 5개 구함.
    - 인기 상품 구함.
  - 저수준
    - DB에서 상세 정보 구함.
    - Daara API에서 상품 5개 구함.
    - 같은 분류에서 속한 상품에서 최근 한 달 판매가 많은 상품 ERP에서 구함.



### Composition over Inheritance

- 상속을 통한 재사용 및 추가 기능을 통한 확장
  - 장점
    - 서브 클래스는 수퍼 클래스의 기능을 재사용
    - 추가적인 기능을 제공
    - 쉽다
  - 단점
    - 변경의 유연함 측면에 치명적 단점
    - 수퍼 클래스의 변경이 다수의 서브 클래스에 영향을 미침
    - 유사한 기능의 확장에서 클래스의 개수가 불필요하게 증가할 수 있다.
      - 2개 이상의 수퍼 클래스의 기능이 필요한 경우
        - 다중상속 불가
        - 1개를 상속받고, 다른 한개는 따로 구현
          - 상속 자체를 잘못 사용할 수 있다.
- **composition**(delegation 위임)
  - 유연성(변경 용이성) 증대
    - polymorphism + composition으로 인해
  - unit test(mock) 용이
  - TDD에 용이
  - Interface의 중요성
- 상속하면 재사용과 상속보다는 조립을 떠올리자



## Functions



### 원칙

- 한가지 일만 해야 한다.
- 밥 아저씨는 극단적으로 함수의 크기는 4줄짜리 여야 한단다.
- indentation, while, nested if 등은 없어야 한다.
- 잘 지어진 서술적인 긴 이름을 갖는 많은/작은 함수들로 유지해야 한다.
  - small many functions
  - nice descriptive long name



### The First Rule of Functions

- 더 이상 작아질 수 없을 만큼 작아야 한다.
- 큰 함수를 보면 클래스로 추출할 생각을 해야한다.
  - Extract Method Object
- 클래스는 일련의 변수들에 동작하는 기능들의 집합



### Fitness Example

- Fitness Example
  - https://github.com/msbaek/fitness-example
  - 메소드명 include - 포함시키다
    - includeSetups()
    - includeTearDowns()
- 개선
  - 읽기 쉬워지고
  - 이해하기 쉬워지고
  - 함수가 자신의 의도를 잘 전달
- 개선의 원인
  - Small
    - 함수의 첫번째 규칙
    - 함수는 작아질 수 있는한 최대한 작아야 한다.
  - 블록이 적어야 함
    - if, else, while 문장 등의 내부 블록은 한줄이어야 한다.
      - try catch 빼고는 괄호가 없어야 한다.
        - 한 줄이면 괄호를 생략 할 수 있으니깐.
      - 함수 호출 일 것
  - Indenting이 적어야 함
    - 함수는 중첩 구조를 갖을 만큼 크면 안된다.
    - 들여쓰기는 한두단계 정도만



### functions should do one thing

- 하나 이상의 일을 하는 함수를 한가지 단순한 일만 하도록 리팩토링
  - 함수의 각 스텝들이 함수 이름이 갖는 추상화 수준보다 한 단계 낮은 것으로만 이루어졌다. 
    - 그렇다면 함수는 한가지 일만 하는 것이다.
- 한가지 일만 하도록 하는 것
  - 원래 코드는 추상화 수준이 다른 많은 단계들을 포함하므로 한가지 이상의 일을 한다는 것이 명확하다.
  - 하지만 의미있게 최종적으로 줄이는 것은 어렵다.
    - surround 메소드의 3줄을 includeSetupAndTeardownsIfTestPage 메소드로 추출
      - 추상화 수준의 변경 없이 코드를 단순히 재인용하게 된다.
  - 함수가 하나 이상의 일을 한다고 말할 수 있는 경우
    - 단순한 구현의 재인용이 아닌 이름으로 함수를 추출할 수 있을 때
- Reading code from top to bottom
  - 함수를 구성하는 원칙
  - 코드를 top-down 이야기체로 읽을 수 있도록 해줌
  - 현재 함수 바로 밑에 현재 함수 다음의 추상화 수준을 갖는 함수들을 배치시킴

```java
public String surround() throws Exception {
    if (ifTestPage())
      	surroundPageWithSetUpsAndTearDowns();
    return pageData.getHtml();
}

private void surroundPageWithSetUpsAndTearDowns() throws Exception {
    content = includeSetups();
    content += pageData.getContent();
    content += includeTearDowns();
    pageData.setContent(content);
}

private boolean ifTestPage() throws Exception {
  	return pageData.hasAttribute("Test");
}
```



### Where do classes go to hide?

- 큰 함수는 실제로는 클래스가 숨어 있는 곳이다.
- 큰 함수는
  - 변수와 인자들
  - 들여쓰기에 존재하고, 변수들을 사용해서 통신하는 기능들의 집합
  - 항상 하나 이상의 클래스로 분리할 수 있다.



### PrimeNumber 예제

- https://github.com/msbaek/print-prime
  - 복잡한 함수를 2개의 클래스로 Extract Method Object하는 과정



### One thing???

- function should do one things, do it well, do it only.
  - 한가지만 해야하고 한가지를 잘해야 하며 오직 한가지만 해야 한다.
- 리팩토링 전의 하나 이상의 일을 하던 메소드
  - caller 입장에서는 one thing
  - reader 입장에서는 one thing이 아니다.
- 하나의 이상의 섹션으로 구성된 함수는 적어도 reader 입장에서는 one thing을 하는 것이 아니다.
  - 코드를 읽을 사람이 가장 중요하고 존중 받아야 한다.
- 큰 함수를 작은 함수들로 쪼갤 때 흥미로운 일을 수행
  - 주요 섹션들을 함수로 추출
  - 함수를 서로 다른 추상화 레벨로 분리
    - 함수가 하나 이상의 추상화 레벨을 다루면 이 함수는 한가지 이상의 일을 하는 것이다.
- 하지만 추상화 레벨은 불분명(fuzzy)하다.
- Extract Till You Drop
  - 함수가 한가지 일만 하는지 어떻게 확실할 수 있는가?
  - 더 이상 extract할 수 없을 때까지 extract하라.
  - extract할 코드를 가진 함수는 한가지 이상의 일을 하는 것이다.
  - 4줄 이내의 함수들로만 구성된 클래스
    - 작은 함수들로 구성되어 있는 클래스는 성격 이 다른 부분이 있다면 클래스 분리하기 쉬워진다.
      - 메소드 그룹핑
  - if, while문 등에서 {}가 보이면 extract 대상
  - {}는 extract할 기회 



### conclusion

- 1st rule
  - function should be small
- 2nd rule
  - smaller than that
- 이름을 잘 지으면 당신뿐 아니라 모든 사람들의 시간을 절약해 준다.
  - 이정표 역할을 하기 때문에
  - 당신의 코드를 이해하기 위한 네비게이션 역할을 한다.
  - 처음부터 좋은 이름이 안나올 수도 있다.
    - 그럴땐 러프하게 만들어 놓고 점점 이름을 명확하게 변경해 나가면 된다.
- 클래스는 큰 함수를 감춘다.
  - 함수를 여러 클래스들에 잘 배분하려면 함수를 작게 만들어야 한다.
  - 함수는 한가지 일만 해야 하고, 한가지만 하는지 확신할 수 있는 유일한 방법은 extract till you drop이다.



## Function Structure



### Arguments

- 인자가 많아지면 복잡도가 증가
- 3개의 인자가 최대
  - 파라미터 객체로 묶기 
- 생성자의 많은 수의 인자를 넘겨야 한다면
  - 이펙티브 자바에 나오는 빌더 패턴 참고.
- Boolean 인자 사용 금지
  - 2가지 이상의 일을 하는 것
    - true, false 2개의 메소드로 분리
- innies not outies
  - 파라미터는 입력으로 작용해야지 출력으로 작용하면 안된다.
  - 메서드로 들어온 인자값을 그대로 수정해서 리턴하지 말자.
    - output argument 대신 return value로 처리
    - 로컬 변수로 빼서 그 로컬 변수를 리턴하는 방식으로
- the null defense
  - null을 전달하거나 기대하는 함수를 만들지 말자.
    - null인 경우의 행위, null이 아닌 경우의 행위
      - 2개의 함수를 만드는 것이 맞다.
  - defensive programming을 지양
    - 코드를 null, 에러 체크로 더럽히지 말자.
    - 팀원이나 단위 테스트를 못 믿는다는 말
  - public api의 경우는 defensive하게 programming한다.



### The Stepdown Rule

- 모든 public은 위에, 모든 private은 아래에
  - public part만 사용자들에게 전달하면 됨.
- 중요한 부분은 위로, 상세한 부분은 밑으로
- 2가지 이점
  - 편집자들은 마지막 부분 제거 가능
  - 독자들은 제일 위에서부터 읽고 지겨우면 땡치면 된다.
- backward reference없이 top에서 bottom으로 읽을 수 있도록



### Switches and cases

- switch 문장 사용을 왜 꺼릴까?

- 객체지향의 가장 큰 이점 중 하나는 의존성 관리 능력

- 모듈 A가 모듈 B의 함수를 사용하는 경우

  - 소스코드 의존성
    - 독립적으로 배포/컴파일/개발 불가

  - 런타임 의존성
    - 실제로 런타임에 A에서 B로 호출이 일어나는 것

- 객체지향이 가능케하는 대단한 것

  - 런타임 의존성은 그대로 둔채로 소스코드 의존성을 역전시킴(Dependency Inversion Principle)
  - 절차
    - 본래의 의존성 제거
    - polymorphic interface를 삽입
    - 모듈 A는 인터페이스에 의존하고, 모듈 B는 인터페이스로부터 derive한다.
      - 기존 A >>> B
        - 변경 A  >>> interface  <<<  B
  - B의 소스코드 의존성은 런타임 의존성과 반대가 된다.
    - 런타임시에 B가 호출 되면서 A가 B를 의존하게 되지만 소스코드에서는 둘은 서로 의존성이 없다.
      - 둘 다 중간에 있는 인터페이스에 의존하고 있다. 
  - Independent Deployability는 객체지향의 강점 중 하나

- switch 문장은 독립적 배포에 방해가 됨

- switch 문장 제거 절차

  - switch 문장을 polymorphic interface 호출로 변환
  - case에 있는 문장들을 별도의 클래스로 추출하여 변경 영향이 발생하지 않도록 한다.

- 실습

  - 마틴 파울러 리팩토링 video store
  - https://github.com/msbaek/videostore
    - remove-switch-statement 브랜치
  - 순서
    - Test부터 Polymorphic하게 수정
      - make it pass
    - 중복 제거
      - Movie 클래스를 생성할 때 마다 생성자에 영화의 타입을 넘기는 부분 중복을 제거 
        - switch문의 분기점이 되는 영화의 타입들을 클래스로 만든다.
          - Movie >>> NewReleaseMovie, ChildrensMovie
    - Push member down 
      - 단축키 이용
      - abstract 메소드로 빼는 부분
        - 변하지 않는 부분은 abstract 클래스 Movie에 변하는 부분은 하위 클래스에서 각각 구현
    - run with coverage
    - remove unused code
    - remove type code

  

### App Partition / Main Partition

- Main Partition
  - Main이 Factory Imp를 만들어서 App Partition에 넘긴다.
  - configuration data 같은 로우 레벨은 여기에 있다. 
  - 스프링이 이 역할을 해준다.
- App Partition
  - 어플리케이션 코드가 존재
  - 인터페이스 타입만 알고 있고 구현체는 Main Partition에서 Main이 생성해서 주입해준다.
- 모듈 다이어그램에서 App 파티션과 Main 파티션 사이에 선을 그을 수 있어야 한다.
- 런타임에는 App 파티션에 Factory가 Main 파티션에 Factory Impl을 호출하니 의존하고 있다고 할 수 있다.
  - 소스코드의 의존성은 Factory Impl이 인터페이스인 Factory를 의존하고 있다. 
- 이러한 기법을 Dependency Injection이라고 한다.



### Temporal Coupling

- 함수들이 순서를 지키며 호출되어야 한다.
  - open, execute, close

- Passing a Block

```java
class FileCommandTemplate {
  	public void process(File file, FileCommand command) {
				// 변경 가능한 부분은 FileCommand 인터페이스로 뺐다.
      	// 이렇게 구현하면 open(), close()를 빼먹을 일이 없다.
      	file.open();
	      command.process(file);
      	file.close();
    }
}

// FileCommandTemplate에 process() 메소드를 호출하면서 파라미터로
// File 객체와 FileCommand 익명클래스를 구현해서 넘긴다.
fileCommandTemplate.process(myfile, new FileCommand() {
  	public void process(File f) {
      	// file processing codes here
    }
});
```



### CQS

- side effect를 관리하는 좋은 방법
- command
  - 시스템의 상태 변경 가능
  - side effect를 갖는다.
  - 아무것도 반환하지 않는다.
- query
  - side effect가 없다.
  - 계산값이나 시스템의 상태를 반환
- CQS 정의
  - 상태를 변경하는 함수는 값을 반환하면 안된다.
  - 값을 반환하는 함수는 상태를 변경하면 안된다.
- 당신 코드의 독자들을 혼란스럽게 하지 마라.
  - 값을 반환하는 함수는 상태를 변경하면 안된다.
  - 상태를 변경하는 함수는 exception을 발생시킬 수는 있지만, 값을 반환할 수는 없다.
  - 약속이고 신뢰다.



### Tell, Don't Ask


- 로그인되었는지 아닌지 아는 것은 user 객체
  - 로그인 여부 상태는 user 객체에 속함
  - 왜 user 상태를 가져다가, user를 대신해서 결정을 하는가?
  - user가 해당 행위를 수행하는 것이 맞다.
- Tell, Don't Ask
  - tell other object what to do
    - 다른 객체가 무엇인가를 하도록 말해라.
  - but not to ask object what the state is.
    - 다른 객체의 상태를 요구해서 가져다가 뭘 하지 말고.

```java
if(user.isLoggedIn()) {
  	user.execute(command);
} else {
  	authenticator.promptLogin();
}
```

```java
try {
  	user.execute(command);
} catch(User.NotLoggedIn e) {
	  annuciator.promptLogin();
}
```

```java
// user 객체가 모든 것을 처리하도록
user.execute(command, annuciator);
```



- 이 규칙을 준수하다보면 query가 많이 없어진다.
  - 이건 매우 좋은 것이다.
    - 왜냐하면 query는 곧 out of control 되는 경향이 있다.
- `o.getX().getY().getZ().doSomething();`
  - Tell, Don't ask의 명확한 위반
  - 뭔가를 tell(doXXX)하기 전에 지속적으로 ask(getXXX)함.
- `o.doSomething();`
  - 이렇게 변경해야 함.
    - o에게 무엇을 하라고 했는데, o는 어떻게 해야 할지 모른다.
    - 하지만 o는 어떤 다른 객체에게 tell하면 되는지는 안다.
    - 최종적으로 처리되기까지 요청이 전파된다.



### Law of Demeter

- 하나의 함수가 전체 시스템의 객체들 간의 네비게이션을 아는 것은 잘못된 설계
  - 함수가 시스템의 전체를 알게 하면 안된다.
  - 개별 함수는 아주 제한된 지식만 가져야 한다.
  - 객체는 요청된 기능 수행을 위해 이웃 객체에게 요청해야 한다.
  - 요청을 수신하면 적절한 객체가 수신할 때까지 전파되어야 한다.
- Law of Demeter는 일련의 규칙을 통해 Tell, Don't Ask를 형식화 한다.
  - 아래와 같은 객체들의 메소드만 호출할 수 있다.
    - 인자로 전달된 객체
    - locally 생성한 객체
    - 필드로 선언된 객체
    - 전역 객체
  - 이전 메소드 호출의 결과로 얻은 객체의 메소드를 호출하면 안된다.
    - `o.getX().getY().getZ().doSomething();`
- ask 대신 tell하면 이웃하는 객체와 디커플링 된다.



### early returns

- `if`에서 할일 다하고 `else`에서 return하는 경우가 있다.
  - 뒤집어서 `else`를 먼저 나오게 해야한다.
    - 조건문을 부정을 만들어서 return이 먼저 나오게
    - 함수를 끝까지 않읽고 최대한 빨리 return을 받고 싶을 수도 있으니깐. 
- 루프의 중간에서 리턴하는 것은 문제
  - break, 루프 중간에서의 return은 loop를 복잡하게함.
  - 코드가 동작하도록 하는 것보다 이해할 수 있게하는 것이 더 중요



### error handling

- stack example
  - https://github.com/msbaek/stack-example
  - 에러 처리를 위해 pop 했을 때 없으면 null을 반환하고, push 했는데 실패 시 false를 반환한다?
    - CQS 위반
    - 이 보다는 exception을 발생시키는 것이 좋다.
  - exception의 이름은 최대한 구체적이어야 한다.
    - Stack.Overflow

```java
public static Stack make(int capacity) {
  	if(capacity < 0)
      	throw new IllegalCapacity();
  	if(capacity == 0)
      	return new ZeroCapacityStack();
  
  	return new BoundedStack(capacity);
}

// RuntimeException인 Stack.IllegalCapacity static inner class
public static class IllegalCapacity extends RuntimeException {
}
```



- checked exception
  - reverse dependency를 유발
    - 하위 클래스에서 어떤 exception이 유발되면 상위 클래스를 변경해야하고
    - 사용되는 모든 코드를 변경해야 함.
  - checked exception을 아예 사용하지 말라.
- 스프링 컨테이너에서 runtime exception이 발생하면 디폴트는 롤백
  - checked exception이 발생하면 롤백이 아니다.
- checked exception을 사용할 때
  - 내가 catch를 해서 명확하게 할 일이 있을 때
    - 익셉션이 발생하면 3초가 슬립 했다가 다시 시도하고 또 안되면 3번 더 시도하고 안되면 로그 남기고 넘어 간다던지
    - 트랜젝션 롤백이나 에러페이지로 보내고 하는건 스프링이 다 해줘서 내가 catch할 이유가 없다.
  - public API의 경우
    - 내가 외부에 API를 제공할 때는 엄격하게 checked exception을 던지자
    - 내가 checked exception을 던지는 public API를 사용하는 경우에는 wrapper를 한번 감싸서 사용한다.
      - wrapper에서 checked exception을 runtime exception으로 바꿔서 던진다.
      - 그래로 쓰면 API를 사용하는 내 모든 코드가 더러워진다. 
- exception들에 어떤 메시지를 담아야 할까?
  - message가 필요 없는 것이 가장 좋다.
  - exception 이름을 정확하게 지어 이름으로 의미가 전달되도록 하라.
    - 가장 좋은 커멘트는 커멘트를 작성하지 않는 것이다.
    - 코드(이름)가 커멘트를 대신하도록 해라.



### Special Cases

- NullObject Pattern(Special Cases Pattern)
  - 특별한 케이스를 추가하기
    - 기존의 코드에서 인터페이스를 추출하고 그 인터페이스를 구현하는 새로운 특별한 케이스를 적용

```java
public class BoundedStack implements Stack {
  
  	...
  
    public static Stack make(int capacity) {
        if(capacity < 0)
            throw new IllegalCapacity();
        if(capacity == 0)
          	// null을 반환하는게 아니라 nullObject를 만들어서 반환
          	// 추출한 인터페이스를 구현하는 새로운 특별한 케이스
            throw new ZeroCapacityStack();

        return new BoundedStack(capacity);
    }

    ...

    private static class ZeroCapacityStack implements Stack {
        ...
    }  
  
  	...
      
}  
```



### Null is not an error

- stack이 empty일 때 top 함수는 무엇을 반환해야 하나?
  - null을 반환할 수도 있다.
    - top을 호출하는 아무도 null을 기대하지는 않을 것이다.
    - null은 NPE를 발생시킬 때 까지 시스템의 여기저기에 조용히 퍼지는 속성이 있다.
  - StackEmpty exception을 발생시키는 것이 좋아 보인다.



### Null is a value

- 이 부분은 패스 null 값은 리턴하지 않는 걸로.



### try도 하나의 역할/기능이다.

- 함수 내에서 try 문장이 있다면 try 문장이 변수 선언을 제외하고는 첫번째 문장이어야 한다.
- try 블록 내에는 한 문장(함수 호출)만 있어야 한다.
- finally가 함수의 마지막 블록이어야 한다.
  - 이우에 어떤 라인도 없어야 한다.
- 함수는 하나의 일만 해야 한다.
  - error handling은 하나의 일이다.



## Form



### Coding Standards

- 조직이 일정 수준의 크기가 되면 관료적인 문서화를 요구
- 필요하다
  - 하지만 별도의 문서는 반대
- Coding Standards는 코드 내에서 명확하게 보여져야 한다.
- 코드가 Coding Standards여야 한다.
- 별도의 문서에 Coding Standards를 작성한다?
  - 코드가 Coding Standards의 예로 적합하지 않다는 것을 의미
  - 코드가 잘되어 있다면 별도의 문서는 필요없다.



### Comments should be Rare

- Coding Standards가 커멘트 작성을 강요하면
  - 프로그래머는 필요해서가 아니라 해야 하므로 커멘트를 작성함.
  - 이런 커멘트는 무의미하다.
  - 이런 커멘트는 무시의 대상이된다.
- 양치기 소년
  - 코드를 보면 뻔히 보이는 무의미한 주석이 많아지면 개발자들이 주석을 잘 안보게 되면서 정작 중요한 주석을 놓치는 경우가 생길 수 있다. 
- comments should be rare.
  - 주석은 특별한 경우에 드물게 작성 되어야 한다.
  - special cases
  - programmer's intent를 위해 반드시 필요할 때
  - 그 커멘트를 읽는 모든 사람들이 감사해야 한다.
    - 쓸데없는 주석으로 읽는 사람의 짜증을 유발해서는 안되겠지?



### Comments are Failures

- 작성자의 의도가 잘 나타나게 프로그램을 작성 한다면 커멘트가 불필요해짐.
- 코드로 의도를 나타낼 수 있을까?
  - Assembly의 경우는 불가
    - 커멘트가 필수적
    - Pascal, Fortran, C도 다소 그런편
    - 커멘트 없이 표현력이 뛰어난 코드를 작성하는 것은 매우 어렵다.
  - Ruby, Java, C# 등은 매우 표현력이 뛰어남.
- 모든 커멘트는 당신의 코드가 잘 표현되고 있지 못하는 것을 나타내는 실패의 상징이다.



### Good Comments

- Legal Comments
- Informative Comments
  - 정규식의 경우
    - `// format matched kk:mm:ss EEE, MMM dd, yyyy`
      - 이런식으로 보기 편하게 
- Warning of Consequences
- TODO Comments
- Public API Documentation



### Bad Comments

- Mumbling
- Redundant Explanations
  - 중족적인 설명
- Mandated Redundancy
  - 자동적으로 만들어지는 커멘트들
- Journal Comments
  - 이제 git이 있는데 수정 내역을 주석으로 남길 필요가 있을까?
- Noways Comments
  - 디폴트 생성자다 뭐 이런 뻔한 커멘트들
- Big Banner Comments
  - 잘보이라고 넣는 배너 커멘트들
- Closing Brace Comments
  - for 문, if 문 끝이 여기다라고 알려주는 주석
- Attribution Comments
  - 누가 추가했는지
- HTML in Comments
  - 커맨트를 HTML로 남기는 경우
  - 소스 diff 보기가 힘들어 진다.
- Non-Local Information
  - 멀리 떨어진 곳의 코드를 설명하는 커멘트는 커멘트와 무관하게 변경될 수 있다.
  - 그런 커멘트를 작성하지 말라.
  - 커멘트를 작성해야만 한다면 커멘트가 설명하는 코드와 밀접한 곳에 작성하라.



### Vertical Formatting

- 공란을 함부로 사용하지 말라.
  - 메소드 사이
  - 변수들 사이
    - private 변수들과 public 변수들 사이
  - 메소드 내
    - 변수 선언과 메소드 실행의 나머지 부분 사이
    - if/while 블록과 다른 고드 사이
- 서로 관련된 것들을 vertical하게 근접해야 함.
  - vertical한 거리가 그들간의 관련성을 나타낸다.
    - 근접해 있는 코드들이 리팩토링 시에 별도의 메소드로 분리 될 수 있다.



### Classes

- 클래스란 무엇인가?
  - private 변수들을 작성함으로써 클래스를 작성한다.
  - 그리고 그 pirvate 변수들을 public 함수들로 조작한다.
  - 외부에서 private 변수들이 없는 것 처럼 보인다.
    - 외부에서 private 변수들이 안보여야는데 getter/setter를 제공하면 너무 잘보이겠지?
- 객체의 상태를 외부에서 사용할 수 있도록 하는 getter/setter/property 등을 제공하는 것은 Bad Design
  - 왜 변수를 private으로 선언하고, getter/setter를 제공하나?
- Tell, Don's Ask
  - 객체가 no observable state를 갖는다면
    - 관찰할 수 없는 상태를 갖는다면
    - 무엇을 하라고 시키기(tell) 쉽고
    - ask할 가치가 없어진다.
  - 이 규칙을 따르는 객체
    - getter가 많지 않다.
    - getter가 많지 않으므로 setter도 많지 않다.
- max cohesive
  - 메소드가 모든 변수를 조작하는 경우
    - 응집도가 높다.
- max cohesive class
  - max cohesive 메소드들로만 구성된 클래스
- getter/setter는 cohesive 하지 않다.
  - 하나의 변수만 사용하기 때문
  - 클래스가 getter/setter를 많이 가질 수록 덜 cohesive해진다.
  - getter/setter가 없어야만 하나?
    - 안 쓸수는 없으니 최소하라.
      - 그래야 cohesion을 최대화할 수 있다.
    - getter를 쓸 때 본래 변수를 그대로 노출하지 말라.
      - 추상화를 통해 제공하라.
        - 이부분의 적용은 고민을 해봐야 겠다.
- Polymorphism이 생각나나?
  - 내부 변수를 hide 했을 때의 이점
  - 상세 구현을 덜 노출할 수록 polymorphic 클래스를 활용할 기회가 늘어남
- Polymorphism은 independent deployability와 plugin structure의 핵심
  - Polymorphism은 CarDriver(클라이언트 코드)를 Car(서버 코드)의 구현 변경으로부터 보호
- 객체지향의 핵심
  - **IoC를 통해 High Level Policy(클라이언트, 비즈니스 로직)를 Low Level Detail로 부터 보호하는 것**
    - CarDriver와 Car의 고수준 정책이 DieselCar, ElectricCar, NuclearCar 등의 저수준 정책에 영향을 받지 않는다.
    - IoC
      - 런타임에는 CarDriver가 Car를 통해 DieselCar에 의존하더라도 소스 코드 의존성은 DieselCar가 Car 인터페이스를 사용하면서 의존하고 있다.



### Data Structures





### Boundaries





### The Impedance Mismatch







## SOLID-Foundation



## SRP



## OCP



## LSP



## ISP



## DIP



## SOLID Case Study





