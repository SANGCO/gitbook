
## 5장 좋은 테스트의 FIRST 속성



### 5.1 FIRST: 좋은 테스트 조건



### 5.2 [F]IRST: 빠르다

- **[F]ast**



### 5.3 F[I]RST: 고립시킨다

- **[I]solate** Your Tests



### 5.4 FI[R]ST: 좋은 테스트는 반복 가능해야 한다

- Good Tests Should Be **[R]epeatable**



### 5.5 FIR[S]T: 스스로 검증 가능하다

- **[S]elf-Validating**



### 5.6 FIRS[T]: 적시에 사용한다

- **[T]imely**



## 6장 Right-BICEP: 무엇을 테스트할 것인가?



### 6.1 [Right]-BICEP: 결과가 올바른가?



### 6.2 Right-[B]ICEP: 경계 조건은 맞는가? 

- **[B]oundary** Conditions



### 6.3 경계 조건에서는 CORRECT를 기억하라



### 6.4 Right-B[I]CEP: 역 관계를 검사할 수 있는가? 

- Checking **[I]nverse** Relationships



### 6.5 Right-BI[C]EP: 다른 수단을 활용하여 교차 검사할 수 있는가? 

- **[C]ross-Checking** Using Other Means



### 6.6 Right-BIC[E]P: 오류 조건을 강제로 일어나게 할 수 있는가? 

- Forcing [**E]rror** Conditions



### 6.7 Right-BICE[P]: 성능 조건은 기준에 부합하는가? 

- **[P]erformance** Characteristics



## 7장 경계 조건: CORRECT 기억법

> 단위 테스트는 종종 경계 조건들에 관계된 결함들을 미연에 방지하는 데 도움이 됩니다. 경계 조건은 행복 경로의 끝에 있는 것으로 자주 문제가 발생합니다.



### 7.1 [C]ORRECT: [C]onformance(준수)

이메일 주소를 예로 들면 @ 기호를 기준으로 기호 앞부분에서 이름을 추출할 수 있다. 이때 null 값, 빈 문자열 반환 등 각 경계 조건이 발생했을 때 어떤 일이 일어나는지 보여 줄 수 있는 테스트 코드를 작성해야 한다. 그리고 데이터에 대한 유효성 검증은 처음 입력될 때 해야 한다. 그러면 뒷단에서는 검사를 생략할 수 있다. 이처럼 시스템의 데이터 흐름을 이해하면 불필요한 검사를 최소화 할 수 있다. 



### 7.2 C[O]RRECT: [O]rdering(순서)

데이터 순서 혹은 커다란 컬렉션에 있는 데이터 한 조각의 위치는 코드가 쉽게 잘못 될 수 있는 CORRECT 조건에 해당한다. ProfilePool 클래스에 ranked() 메소드는 Collections.sort를 이용하고 있는데 오름차순, 내림차순으로 정렬되는 부분을 좀 더 공부해봐야겠다.



### 7.3 CO[R]RECT: [R]ange(범위)

불변성이 깨지는 테스트를 만들고 @ExpectedToFail, @Ignore 어노테이션을 붙여준다. ~~JUnit5를 사용중인데 @ExpectedToFail을 어떤 라이브러리에서 가지고 와서 써야 할지 감을 못잡고 있음.~~ JUnit4에 @Ignore는 JUnit5에서는 @Disabled. @ExpectedToFail은 커스텀 어노테이션이고 목적은 주석용, 실패를 의도한 테스트라는 거지. @Disabled는 실패가 예상되는 테스트에 커서를 두고 실행을 시키면 Fail이 나고 전체 테스트를 돌리면 무시가 된다.

#### 7.3.1 불변성을 검사하는 사용자 정의 매처 생성

불변성을 검사하는 매처를 만들어서 @After 메서드에서 사용하는 방법.

#### 7.3.2 불변 메서드를 내장하여 범위 테스트

클래스 내에 불변성을 검사하는 메서드를 만들어서 테스트 시에 호출.



### 7.4 COR[R]ECT: [R]eference(참조)

shift를 할 때 필수적으로 전제가 되야하는 조건에 대해 테스트를 한다. (ex. 고객의 계정 히스토리를 달라는 요청은 해당 고객이 로그인을 했다는 전제가 있어야한다. ) preconditions를 // given으로 주고 // when 상황 부여 // then postconditions나 side effects를 테스트의 단언으로 명시.

- 어떤 메서드를 테스트할 때는 다음을 고려해야 한다.
  - 범위를 넘어서는 것을 참조하고 있지 않은지
  - 외부 의존성은 무엇인지
  - 특정 상태에 있는 객체를 의존하고 있는지 여부
  - 반드시 존재해야 하는 그 외 다른 조건들




### 7.5 CORR[E]CT: [E]xistence(존재)

메서드가 홀로 설 수 있게 만들자. 호출한 메소드가 null을 반환하거나, 기대하는 파일이 없거나, 네트워크가 다운되는 등 어떤 일이 일어나는지 확인하는 테스트를 작성하자. 



### 7.6 CORRE[C]T: [C]ardinality(기수)

개수의 개념은 테스트할 작업 목록을 도출하는 데 도움을 준다. 테스트 코드는 0, 1, n이라는 경계 조건에만 집중하고 n은 비즈니스 요구 사항에 따라 바뀔 수 있다. 요구 사항이 바뀌어도 변경되는 지점은 한군데로. `public static final int MAX_ENTRIES = 20`



### 7.7 CORREC[T]: [T]ime(시간)

- 시간에 관하여 마음에 담아 두어야 할 측면
  - 상대적 시간(시간 순서)
  - 절대적 시간(측정된 시간)
  - 동시성 문제들



## 8장 깔끔한 코드로 리팩토링하기

> 낮은 중복성과 높은 명확성이라는 두 가지 목표를 합리적인 비용과 놀라운 투자 수익률(ROI)로 달성할 수 있습니다. 좋은 소식은 단위 테스트를 만들면 이러한 목표에 도달할 수 있다는 것입니다.



### 8.1 작은 리팩토링

테스트는 리팩토링을 통해 코드를 이리저리 옮겨도 시스템이 정상 동작함을 확인할 수 있는 보호 장치이다.

#### 8.1.1 리팩토링의 기회

matches() 메서드는 상당히 빽빽하고 꽤 많은 로직을 담고 있다.

#### 8.1.2 메서드 추출: 두 번째로 중요한 리팩토링 친구

matches() 메서드의 복잡도를 줄여 코드가 무엇을 담당하는지 쉽게 이해하게 만들었다. 조건문이 복잡하면 잘 읽히지 않는다. 복잡한 조건문을 별도의 메서드로 분리.



### 8.2 메소드를 위한 더 좋은 집 찾기

새로운 메소드를 추출 했다면 해당 메소드가 현재의 클래스에 있는게 맞는지 생각해본다.



### 8.3 자동 및 수동 리팩토링

matches() 메서드에서 어떤 프로파일이 매치되는지 여부를 결정하는 코드를 찾아 anyMatches() 라는 메서드로 분리 시킨다. --~~리팩토링 결과 똑같은 for문이 2번 돌고 있다. 이거 괜찮나?~~ 



### 8.4 과한 리팩토링?

기존 메소드에서 새로운 메서드 3개를 도출 했다. 이제 for문이 3개다. 흠 코드가 가독성도 좋고 각각의 세부적인 부분을 테스트 하기 용이하긴한데...



```java
  // Before
  public boolean matches(Criteria criteria) {
    score = 0;

    boolean kill = false;
    boolean anyMatches = false;
    for (Criterion criterion: criteria) {   
      Answer answer = answers.get(
        criterion.getAnswer().getQuestionText()); 
      boolean match = 
        criterion.getWeight() == Weight.DontCare || 
        answer.match(criterion.getAnswer());

      if (!match && criterion.getWeight() == Weight.MustMatch) {  
        kill = true;
      }
      if (match) {         
        score += criterion.getWeight().getValue();
      }
      anyMatches |= match;
    }
    if (kill)       
      return false;
    return anyMatches;
  }
```

```java
  // After
	public boolean matches(Criteria criteria) {
    calculateScore(criteria);
    if (doesNotMeetAnyMustMatchCriterion(criteria))
      return false;
    return anyMatches(criteria);
  }

  private boolean doesNotMeetAnyMustMatchCriterion(Criteria criteria) {
    for (Criterion criterion: criteria) {
      boolean match = criterion.matches(answerMatching(criterion));
      if (!match && criterion.getWeight() == Weight.MustMatch) 
        return true;
    }
    return false;
  }

  private void calculateScore(Criteria criteria) {
    score = 0;
    for (Criterion criterion: criteria) 
      if (criterion.matches(answerMatching(criterion))) 
        score += criterion.getWeight().getValue();
  }

  private boolean anyMatches(Criteria criteria) {
    boolean anyMatches = false;
    for (Criterion criterion: criteria) 
      anyMatches |= criterion.matches(answerMatching(criterion));
    return anyMatches;
  }
```



## 9장 더 큰 설계 문제

> 특히 단일 책임 원칙(SRP)에 초점을 맞추는데, 이는 좀 더 작은 클래스를 만들어 무엇보다 유연성과 테스트 용이성을 높여 줍니다. 그리고 명령-질의 분리도 알아보는데, 이것은 부작용을 만들고 동시에 값을 반환하여 사용자를 기만하는 메서드를 만들지 않도록 합니다.



### 9.1 Profile 클래스와 SRP

- 단일 책임의 이점
  - 변경으로 인한 리스크가 줄어든다.
    - 클래스에 많은 책임을 부여할 수록 코드 변경시에 다른 동작들이 깨지기 쉽다.
  - 재활용
    - 더 작고 집중화된 클래스는 다른곳에서 재사용하기 용이하다.



### 9.2 새로운 클래스 추출

for문도 그렇고, 같은 데이터를 중복적으로 가지고 있는 새로운 클래스의 도출도 중복이라 피해야 한다고 생각했는데 나는 가독성, 단일 책임 원칙 등을 너무 등한시 한건 아닌가 싶다. 유지 보수에 용이한 코드라는 측면에서 새로운 클래스를 적극 도출 해봐야겠다. 향후에 리팩토링을 하다보면 데이터를 중복적으로 가지고 있는 부분은 사라진다.



Profile 클래스에서 Criteria는 필요없어서 변수를 제거하고 matches() 메소드의 매개변수로 받아서 바로 MatchSet 클래스의 생성자로 던져버린다. Profile 클래스는 MatchSet을 가지고 있지 않다. 메소드 내에서 생성해 버린다.



### 9.3 명령-질의 분리

어떤 값을 반환하고 시스템에 있는 어떤 클래스나 엔터티의 상태를 변경하는 메서드는 명령-질의 분리 원칙 위반이다. 부작용을 생성하는 작업을 하거나 질의에 대한 대답을 하거나 둘 중 하나만 해야지 둘 다 해서는 안된다. 이번 파트에서는 Profile 클래스에서 점수도 걷어내 버리고 매칭과 점수 등에 관한 요청을 받으면 MatchSet을 리턴하게 했다. MatchSet을 반환 받은 클라이언트는 스스로 점수를 얻거나 조건에 매칭되는지 여부를 알 수 있다. 기존에 boolean 값을 리턴하면서 상태값을 바꾸는 형태를 수정. ~~근데 새로 도출한 클래스는 따로 테스트 클래스를 안만들어도 되나? 예제에는 없던데. Profile을 통해 MatchSet을 충분히 테스트하면 되는건가?~~ 



### 9.4 단위 테스트의 유지 보수 비용

- 리팩토링을 하면 테스트가 깨질 수도 있는데 이걸 고치는 노력은 단위 테스트를 소유하는 비용이다.

- 보통 돌아오는 가치가 훨씬 크기 때문에 깨진 테스트 코드를 고치는 비용을 받아들인다.
  - 결함이 거의 없는 코드를 갖는 이점
  - 다른 코드가 깨질 것을 걱정하지 않으면서 코드를 변경할 수 있는 이점
  - 코드가 정확히 어떻게 동작하는지 알 수 있는 이점
- 실패하는 테스트의 정도는 부정적인 설계의 지표?
  - 더 많은 테스트가 동시에 깨질수록 더욱더 많은 설계 문제가 있을 것.

- public 메소드 테스트가 private 메소드 테스트를 포함.
  - private 메소드를 테스트하려는 충동은 클래스가 필요 이상으로 커졌다는 또 다른 힌트
  - private 메소드가 자꾸 늘어나면 내부 동작을 새 클래스로 옮기고 public으로 만드는 것이 좋다.



### 9.5 다른 설계에 관한 생각들

- MatchSet 클래스 생성자에서 점수를 계산하는 부분을 걷어내기
  - score 변수를 메서드 안으로 옮겨서 요청이 들어오면 계산을 해서 리턴
  - getScore() 메서드가 호출될 때마다 매번 점수를 계산하는 것이 성능 저하로 이어진다면?
    - 지연 초기화(lazy initialization) 
    - score를 다시 인스턴스 변수로 올리고 요청이 들어왔을 때 score가 초기화가 안되어 있을때만 점수 계산해서 리턴

- 중복된 반복문에 방문자 패턴을 고려
  - 비지터 패턴을 적용하면 for문은 한번 도는데 그 각각의 결과값을 저장할 변수들을 만들어야 되는거 아닌가?

```java
interface CarElement {
    void accept(CarElementVisitor visitor);
}

interface CarElementVisitor {
    void visit(Body body);
    void visit(Car car);
    void visit(Engine engine);
    void visit(Wheel wheel);
}

class Wheel implements CarElement {
  private final String name;

  public Wheel(final String name) {
      this.name = name;
  }

  public String getName() {
      return name;
  }

  @Override
  public void accept(CarElementVisitor visitor) {
      visitor.visit(this);
  }
}

class Body implements CarElement {
  @Override
  public void accept(CarElementVisitor visitor) {
      visitor.visit(this);
  }
}

class Engine implements CarElement {
  @Override
  public void accept(CarElementVisitor visitor) {
      visitor.visit(this);
  }
}

class Car implements CarElement {
    private final List<CarElement> elements;

    public Car() {
        this.elements = List.of(
            new Wheel("front left"), new Wheel("front right"),
            new Wheel("back left"), new Wheel("back right"),
            new Body(), new Engine()
        );
    }

    @Override
    public void accept(CarElementVisitor visitor) {
        for (CarElement element : elements) {
            element.accept(visitor);
        }
        visitor.visit(this);
    }
}

class CarElementDoVisitor implements CarElementVisitor {
    @Override
    public void visit(Body body) {
        System.out.println("Moving my body");
    }

    @Override
    public void visit(Car car) {
        System.out.println("Starting my car");
    }

    @Override
    public void visit(Wheel wheel) {
        System.out.println("Kicking my " + wheel.getName() + " wheel");
    }

    @Override
    public void visit(Engine engine) {
        System.out.println("Starting my engine");
    }
}

class CarElementPrintVisitor implements CarElementVisitor {
    @Override
    public void visit(Body body) {
        System.out.println("Visiting body");
    }

    @Override
    public void visit(Car car) {
        System.out.println("Visiting car");
    }

    @Override
    public void visit(Engine engine) {
        System.out.println("Visiting engine");
    }

    @Override
    public void visit(Wheel wheel) {
        System.out.println("Visiting " + wheel.getName() + " wheel");
    }
}

public class VisitorDemo {
    public static void main(final String[] args) {
        Car car = new Car();

        car.accept(new CarElementPrintVisitor());
        car.accept(new CarElementDoVisitor());
    }
}  
```



