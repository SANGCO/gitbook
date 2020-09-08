

## OOP



## Functions



## Function Structure

- Arguments
  - 인자가 많아지면 복잡도가 증가
  - 3개의 인자가 쵀대
    - Introduce Parameter Object
      - 파라미터 객체로 묶기 
  - 생성자의 많은 수의 인자를 넘겨야 한다면
    - 이펙티브 자바에 나오는 빌더 패턴 참고.
  - Boolean 인자 사용 금지
    - 2가지 이상의 일을 하는 것
      - 2개의 메소드로 분리
  - innies not outies
    - output argument 대신 return value로 처리해라.
      - 메서드로 들어온 인자값을 그대로 수정해서 리턴하지 말자.
        - output argument 대신 return value로 처리
        - 로컬 변수로 빼서 그 로컬 변수를 리턴하는 방식으로
  - the null defense
    - null을 전달/기대하는 함수를 만들지 말자.
      - null인 경우의 행위 + null이 아닌 경우의 행위
        - 2개의 함수를 만드는 것이 맞다.
    - defensive programming을 지양
      - 코드를 null, 에러 체크로 더럽히지 말자.
      - 팀원이나 단위 테스트를 못 믿는다는 말
    - public api의 경우는 defensive하게 programming한다.



- The Stepdown Rule
  - 모든 public은 위에, 모든 private은 아래에
  - public part만 사용자들에게 전달하면 됨.
  - 2가지 이점
    - 편진자들은 마지막 부분 제거 가능
    - 독자들은 제일 위에서부터 읽고 지겨우면 땡치면 된다.
  - backward reference없이 top에서 bottom으로 읽을 수 있게



- Switches and cases

  - switch 문장 사용을 왜 꺼리나?

  - 객체지향의 가장 큰 이점 중 하나는 의존성 관리 능력이다.

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
    - Independent Deployability는 객체지향의 강점 중 하나

  - switch 문장은 독립적 배포에 방해가 됨

  - switch 문장 제거 절차

    - switch 문장을 polymorphic interface 호출로 변환
    - case에 있는 문장들을 별도의 클래스로 추출하여 변경 영향이 발생하지 않도록 한다.

  - 실습

    - 마틴 파울러 리팩토릴 video store
    - Push member down 단축키
      - abstract 메소드로 빼는 부분

  - application/main partition



- Temporal Coupling
- CQS
- Tell Don't Ask
- Law of Demeter
- early returns
- error handling
- Special Cases
- Null is not an error
- Null os value
- try도 하나의 역할/기능이다.



​	