

## 2장 문자열 계산기 구현을 통한 테스트와 리팩토링



#### 2.1 main() 메소드를 활용한 테스트의 문제점

- main() 메소드 하나에서 프로덕션 코드에 여러 메소드들 한번에 테스트
  - 테스트 클래스를 별도의 클래스로 분리 했지만 갈수록 테스트용 main 메소드의 복잡도는 증가할 것이다.
- 메소드 분리를하고 메인 메소드에서 호출
  - 이렇게 해도 main() 모든 기능을 한번에 테스트해야함.
    - 한 기능만 테스트하려면 테스트 안 할 메소드 호출하는 부분을 주석처리 해야함.
      - 역시 허접함. 
- 심지어 main() 메소드로 테스트하는 방식은 수동이다.
- 이러한 문제를 해결하기 위해 JUnit 등장



#### 2.2 JUnit을 활용해 main() 메소드 문제점 극복

- 한 번에 메소드 하나에만 집중
- 결과 값을 눈이 아닌 프로그램을 통해 자동화
  - assertEquals()
- 테스트 코드 중복 제거
  - 개발자가 가져야 할 좋은 습관 중의 하나는 중복 코드를 제거하는 것
    - 중복 코드는 프로그래밍의 가장 큰 적 중의 하나
  - @Before
    - 테스트 메소드 간에 서로 영향을 미치지 않고 독립적으로 메소들르 실행하기 위해 쓰인다.
    - 각 테스트가 실행될 때마다 인스턴스가 매번 다시 생성되는 방식으로 동작한다.



#### 2.3 문자열 계산기 요구사항 및 실습

- 요구사항 분리 및 각 단계별 힌트

```java
// 힌트 "//"와 "\n" 문자 사이에 커스텀 구분자를 지정할 수 있다.
Matcher m = Pattern.compile("//(.)\n(.*)").matcher(text);
if (m.find()) {
  	String customDelimeter = m.grout(1);
  	String[] tokens = m.group(2).split(customDelimeter);
}
```



- 추가 요구사항
  - 요구사항을 만족하는 코드를 구현했다고 개발이 끝난 건 아니다.
  - 소스코드를 구현했으면 반드시 리팩토링이 뒤따라야 한다.
    - 리팩토링
      - 중복을 제거
      - 읽기 좋은 코드를 구현하기 위해 구조를 변경
      - 소스코드의 가독성을 높이고 유지보수를 편하게 하기 위해 소스코드의 구조를 변경하는 것.
      - 리팩토링을 한다고 기능상의 결과가 변경되면 안된다.
    - 초보자를 위한 리팩토링 3가지 원칙
      - 메소드가 한 가지 책임만 가지도록 구현한다.
      - 인덴트(indent, 들여쓰기) 깊이를 1단계로 유지한다.
      - else를 사용하지 마라.



#### 2.4 테스트와 리팩토링을 통한 문자열 계산기 구현

- 앞 챕터를 보면 요구사항이 단순해도 소스코드의 복잡도가 금방 증가한다.
  - 복잡도가 증가하면 새로운 요구사항을 추가 구현하기 힘들다.
  - 테스트가 째질 경우 디버징하기도 힘들다.
- 복잡도가 높은 요구사항을 구현할 때 높아지는 소스코드 복잡도는 우짤것인가
  - **복잡도를 낮출 수 있는 방법 중의 하나가 끊임없는 리팩토링을 통해 소스코드를 깔끔하게 구현하는 연습을 하는 것이다.**

- 요구사항을 작은 단위로 나누기
  - 복잡한 문제를 풀어가기 위해 첫 번째로 진행해야 하는 작업이 복잡한 문제를 작은 단위로 나눠 좀 더 쉬운 문제로 만드는 작업이다.

- 모든 단계의 끝은 리팩토링
  - 소스코드의 복잡도가 쉽게 증가하는 이유는 하나의 요구사항을 완료한 후 리팩토링을 하지 않은 상태에서 다음 단계로 넘어가기 때문이다.
    - 각 단계에서 다음 단계로 넘어가기 위한 작업의 끝은 내가 기대하는 결과를 확인했을 때가 아니라 결과를 확인한 후 리팩토링까지 완료했을 때이다.
  - 지금까지 개발 과정 "구현 - > 테스트를 통한 결과 확인"
    - 앞으로 "구현 - > 테스트를 통한 결과 확인 - > 리팩토링"
- 문자열 계산기 구현
  - private으로 분리한 메소드들 말고 public으로 공개하고 있는 add() 메소드가 얼마나 읽기 쉽고, 좋은지가 가장 중요하다. 
  - 세부 구현은 모두 private 메소드로 분리해 일단 관심사에서 제외
    - add() 메소드를 딱 보면 무슨 일을 하는지 전체 흐름이 쉽게 파악된다.

```java
public class StringCalculator {
  
  	public int add(String text) {
      	if (isBlank(text)) {
          	return 0;
        }
      	
      	return sum(toInts(split(text)));
    }
  
  	private boolean isBlank(String text) {
      	return text == null || text.isEmpty();
    }
  
  	private String[] split(String text) {
      	return text.split(",");
    }
  
  	private int[] toInts(String[] values) {
      	int[] numbers = new int[values.length];
      	for (int i = 0; i < values.length; i++) {
          	numbers[i] = Integer.parseInt(values[i]);
        }
      	return numbers;
    }
  
  	private int sum(int[] numbers) {
      	int sum = 0;
      	for (int number : numbers) {
          	sum += number;
        }
    }
  
}
```





#### 2.5 추가 학습 자료

- 테스트 주도 개발(Test Driven Development, 이하 TDD)과 리팩토링
- 정규 표현식
  - 손에 잡히는 정규 표현식
  - [온라인 상에서 연습할 수 있는 곳](https://regexr.com/)





## 3장 개발 환경 구축 및 웹 서버 실습 요구사항

#### 3.1 서비스 요구사항

#### 3.2 로컬 개발 환경 구축

#### 3.3 원격 서버에 배포

#### 3.4 웹 서버 실습

#### 3.5 추가 학습 자료

## 4장 HTTP 웹 서버 구현을 통해 HTTP 이해하기

#### 4.1 동영상을 활용한 HTTP 웹 서버 실습

#### 4.2 HTTP 웹 서버 구현

#### 4.3 추가 학습 자료









## 5장 웹 서버 리팩토링, 서블릿 컨테이너와 서블릿의 관계

#### 5.1 HTTP 웹 서버 리팩토링 실습

#### 5.2 웹 서버 리팩토링 구현 및 설명

#### 5.3 서블릿 컨테이너, 서블릿/JSP를 활용한 문제 해결

#### 5.4 추가 학습 자료

## 6장 서블릿/JSP를 활용해 동적인 웹 애플리케이션 개발하기

#### 6.1 서블릿/JSP로 회원관리 기능 다시 개발하기

#### 6.2 세션(HttpSession) 요구사항 및 실습

#### 6.3 세션(HttpSession) 구현

#### 6.4 MVC 프레임워크 요구사항 1단계

#### 6.5 MVC 프레임워크 구현 1단계

#### 6.6 쉘 스크립트를 활용한 배포 자동화

#### 6.7 추가 학습 자료


## 7장 DB를 활용해 데이터를 영구적으로 저장하기

#### 7.1 회원 데이터를 DB에 저장하기 실습

#### 7.2 DAO 리팩토링 실습

#### 7.3 동영상을 활용한 DAO 리팩토링 실습

#### 7.4 DAO 리팩토링 및 설명

#### 7.5 추가 학습 자료

## 8장 AJAX를 활용해 새로고침 없이 데이터 갱신하기

#### 8.1 질문/답변 게시판 구현

#### 8.2 AJAX 활용해 답변 추가, 삭제 실습

#### 8.3 MVC 프레임워크 요구사항 2단계

#### 8.4 MVC 프레임워크 구현 2단계

#### 8.5 추가 학습 자료