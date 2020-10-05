

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



### Tell Don't Ask


- 로그인되었는지 아닌지 아는 것은 user 객체
  - 로그인 여부 상태는 user 객체에 속함
  - 왜 user 상태를 가져다가, user를 대신해서 결정을 하는가?
  - user가 해당 행위를 수행하는 것이 맞다.
- Tell Don't Ask
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







## SOLID-Foundation



## SRP



## OCP



## LSP



## ISP



## DIP



## SOLID Case Study





