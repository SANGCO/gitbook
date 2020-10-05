

## OOP



## Functions



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
    - 실제로 런타임에 A에거 B로 호출이 일어나는 것

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



### CQS



### Tell Don't Ask



### Law of Demeter



### early returns



### error handling



### Special Cases



### Null is not an error



### Null os value



### try도 하나의 역할/기능이다.





	## Form





## SOLID-Foundation



## SRP



## OCP



## LSP



## ISP



## DIP



## SOLID Case Study





