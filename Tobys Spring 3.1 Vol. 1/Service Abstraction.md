- 지금까지 만든 DAO에 트랜잭션을 적용 하면서
  - 성격이 비슷한 여러 종류의 기술을 추상화
    - 일관된 방법으로 사용할 수 있도록 지원



## 5.1 사용자 레벨 관리 기능 추가



### 5.1.1 필드 추가



#### Level 이늄



#### User 필드 추가



#### UserDaoTest 테스트 수정



#### UserDaoJdbc 수정

- DB까지 연동되는 테스트를 잘 만들어 두면 SQL 문장에 오타도 빠르게 잡아낼 수 있다.
  - 테스트가 없다면 수동 테스트를 통해 에러를 발결하게 될 것이다. 
  - 그때까지 진행한 빌드와 서버 배치, 서버 재시작, 수동 테스트 등에 소모한 시간은 낭비에 가깝다.
  - 빠르게 실행 가능한 포괄적인 테스트를 만들어두면 이렇게 기능의 추가나 수정이 일어날 때 그 위력을 발휘한다.



### 5.1.2 사용자 수정 기능 추가



#### 수정 기능 테스트 추가



#### UserDao와 UserDaoJdbc 수정



#### 수정 테스트 보완

- UPDATE 문장에서 WHERE 절을 빼먹는 경우
  - 수정하지 않아야 할 로우의 내용을 바꿔버리는 문제
    - 리턴 값으로 확인
    - 원하는 사용자 외의 정보가 변경되지는 않았는지 직접 확인



### 5.1.3 UserService.upgradeLevels()

- DAO는 데이터를 어떻게 가져오고 조작할지를 다루는 곳
  - 비즈니스 로직을 담을 클래스를 하나 추가
    - UserService
    - UserDao의 구현 클래스가 바뀌어도 영향을 받지 않도록 UserService에서는 UserDao 인터페이스 타입을 사용



#### UserService 클래스와 빈 등록



#### UserServiceTest 테스트 클래스



#### upgradeLevels() 메소드



#### upgradeLevels() 테스트



### 5.1.4 UserService.add()



### 5.1.5 코드 개선



#### upgradeLevels() 메소드 코드의 문제점

- upgradeLevels() 메소드 for 루프 안에 if/elseif/else 블록
  - 읽기 불편하다.
  - 성격이 다른 여러 가지 로직이 한데 섞여있다.
  - 상당히 변화에 취약하고 다루기 힘든 코드



#### upgradeLevels() 리팩토링

- 레벨을 업그레이드하는 작업의 기본 흐름만 `리스트 5-23`과 같이 먼저 만들어보자.
  - 구체적인 구현에서 외부에 노출할 인터페이스를 분리하는 것과 마찬가지 작업이라고 생각하면 된다.

- 업그레이드 가능한지 여부를 확인하는 메소드와 레벨 업그레이드 작업 메소드로 분리
  - canUpgradeLevel()
    - switch문으로 레벨을 구분하고, 각 레벨에 대한 업그레이드 조건을 만족하는지를 확인
  - upgradeLevel()
- enum Level이 업그레이드 순서를 담고 있도록 수정
- 사용자 정보가 바뀌는 부분을 UserService 메소드에서 User로 옮긴다.
  - UserService가 일일이 레벨 업그레이드시에 User의 어떤 필드를 수정한다는 로직을 갖고 있기보다는, User에게 레벨 업그레이드를 해야 하니 정보를 변경하라고 요청하는 편이 낫다.

- 개선한 코드를 살펴보면 각 오브젝트와 메소드가 각각 자기 몫의 책임을 맡아 일을 하는 구조
  - 각자 자기 책임에 충실한 작업만을 하고 있으니 코드를 이해하기도 쉽다.
  - 변경이 필요할 때 어디를 수정해야 할지도 쉽게 알 수 있다.
  - 각각을 독립적으로 테스트하도록 만들면 테스트 코드도 단순해진다.

- 객체지향적인 코드는 다른 오브젝트의 데이터를 가져와서 작업하는 대신 데이터를 갖고 있는 다른 오브젝트에게 작업을 해달라고 요청한다.
  - 오브젝트에게 데이터를 요구하지 말고 작업을 요청하라는 것이 객체지향 프로그래밍의 가장 기본이 되는 원리



#### User 테스트



#### UserServiceTest 개선

```java
// checkLevelUpgraded 메소드 체크

@Test
public void upgradeLevels() {
  	...
      
    checkLevelUpgraded(users.get(0), false);
    checkLevelUpgraded(users.get(1), true);
  
  	...
}

private void checkLevelUpgraded(User user, boolean upgraded) {
		User userUpdate = userDao.get(user.getId());
		if (upgraded) {
		  	assertThat(userUpdate.getLevel(), is(user.getLevel().nextLevel()));
		}
		else {
				assertThat(userUpdate.getLevel(), is(user.getLevel()));
		}
}
```

- 테스트와 애플리케이션 코드에 나타난 숫자의 중복 제거는 당연하다.
  - 한 가지 변경 이유가 발생했을 때 여러 군데를 고치게 만든다면 중복이기 때문이다.
  - 정수형 상수로 변경
    - UserServiceTest에서도 UserService에서 정의해둔 상수를 import해서 사용하게 변경

- UserService를 인터페이스로
  - 레벨을 업그레이드하는 정책을 유연하게 변경할 수 있도록 개선
  - 연말 이벤트나 새로운 서비스 홍보기간 중에는 레벨 업그레이드 정책을 다르게 적용할 필요가 있을 수도 있다.
    - UserService를 인터페이스로 만들고 새로운 업그레이드 정책을 담은 클래스 따로 만들어서 DI
    - 이벤트가 끝나면 기존 업그레이드 정책 클래스로 다시 변경



## 5.2 트랜잭션 서비스 추상화

> "정기 사용자 레벨 관리 작업을 수행하는 도중에 네트워크가 끊기거나 서버에 장애가 생겨서 작업을 완료할 수 없다면, 그때까지 변경된 사용자의 레벨은 그대로 둘까요? 아니면 모두 초기 상태로 되돌려 놓아야 할까요?"



### 5.2.1 모 아니면 도



#### 테스트용 UserService 대역

- 테스트 클래스 내부에 스태틱 클래스로 만들기
  - UserService를 상속한 TestUserService 클래스
  - TestUserServiceException



#### 강제 예외 발생을 통한 테스트



#### 테스트 실패의 원인

- 하나의 트랜잭션 안에서 동작하지 않았기 때문



### 5.2.2 트랜잭션 경계설정



#### JDBC 트랜잭션의 트랜잭션 경계설정

- JDBC에서 트랜잭션을 시작하려면 자동커밋 옵션을 false로 만들어주면 된다.
- 트랜잭션의 경계설정(transaction demarcation)
  - setAutoCommit(false)로 트랜잭션의 시작을 선언하고 commit() 또는 rollback()으로 트랜잭션을 종료하는 작업
- 트랜잭션의 경계는 하나의 Connection이 만들어지고 닫히는 범위 안에 존재한다는 점도 기억해두자.
  - 하나의 DB 커넥션 안에서 만들어지는 트랜잭션을 로컬 트랜잭션이라고도 한다.



#### UserService와 UserDao의 트랜잭션 문제

- DAO 메소드를 호출할 때마다 하나의 새로운 트랜잭션이 만들어지는 구조



#### 비즈니스 로직 내의 트랜잭션 경계설정

- 리스트 `5-28` upgradeLevels의 트랜잭션 경계설정 구조
  - DB Connection을 upgradeLevels() 메소드 내에서 생성하고 종료 시켜야한다.
- UserDao에 메소드를 호출 할 때 파라미터로 Connection 객체 넣어주기 



#### UserService 트랜잭션 경계설정의 문제점

- DB 커넥션을 비롯한 리소스의 깔끔한 처리를 가능하게 했던 JdbcTemplate을 더 이상 활용할 수 없다.
  - try/catch/finally
- DAO의 메소드와 비즈니스 로직을 담고 있는 UserService의 메소드에 Connection 파라미터가 추가돼야 한다.
  - 싱글톤
  - 멀티스레드 환경
- Connection 파라미터가 UserDao 인터페이스 메소드에 추가되면 UserDao는 더 이상 데이터 액세스 기술에 독립적일 수가 없다.
  - EntityManager
  - Session
- DAO 메소드에 Connection 파라미터를 받게 하면 테스트 코드에도 영향을 미친다.



### 5.2.3 트랜잭션 동기화

- 지금까지 만든 깔끔하게 정리된 코드를 포기해야 할까? 아니면, 트랜잭션 기능을 포기해야 할까?
  - 물론 둘 다 아니다. 스프링은 이 딜레마를 해결할 수 있는 멋진 방법을 제공해준다. 



#### Connection 파라미터 제거

- 독립적인 트랜잭션 동기화 방식
  - Connection 오브젝트를 특별한 저장소에 보관
  - `그림 5-3` 트랜잭션 동기화를 사용한 경우의 작업 흐름



#### 트랜잭션 동기화 적용

- `DataSourceUtils.getConnection(dataSource);`
  - DB 커넥션 생성과 동기화를 함께 해주는 유틸리티 메소드

- `TransactionSynchronizationManager`
  - 스프링이 제공하는 트랜잭션 동기화 관리 클래스
  - 먼저 이 클래스를 이용해 트랜잭션 동기화 작업을 초기화
- DataSourceUtils의 getConnection() 메소드는 Connection 오브젝트를 생성해줄 뿐만 아니라 트랜잭션 동기화에 사용하도록 저장소에 바인딩 해준다.



#### 트랜잭션 테스트 보완



#### JdbcTemplate과 트랜잭션 동기화

- JdbcTemplate이 제공해주는 세 가지 유용한 기능
  - try/catch/finally 작업 흐름 지원
  - SQLException의 예외 변환
  - JdbcTemplate을 사용하는 메소드에서 트랜잭션 동기화를 시작해놓았다면 트랜잭션 동기화 저장소에 들어 있는 DB 커넥션을 가져와서 사용
    - 미리 생성돼서 트랜잭션 동기화 저장소에 등록된 DB 커넥션이나 트랜잭션이 없는 경우에는 JdbcTemplate이 직접 DB 커넥션을 만들고 트랜잭션을 시작해서 JDBC 작업을 진행



### 5.2.4 트랜잭션 서비스 추상화



#### 기술과 환경에 종속되는 트랜잭션 경계설정 코드

- 새로운 고객 G 사
  - 여러개의 DB를 사용
  - 하나의 트랜잭션 안에서 여러 개의 DB에 데이터를 넣는 작업
- JDBC의 Connection은 로컬 트랜잭션
  - 하나의 DB Connection에 종속된다.
- 글로벌 트랜잭션
  - 각 DB와 독립적으로 만들어지는 Connection을 통해서가 아니라, 별도의 트랜잭션 관리자를 통해 트랜잭션을 관리
  - 글로벌 트랜잭션을 적용해야 트랜잭션 매니저를 통해 여러 개의 DB가 참여하는 작업을 하나의 트랜잭션으로 만들 수 있다.
- JTA (Java Transaction API)
  - 글로벌 트랜잭션을 지원하는 트랜잭션 매니저를 지원하기 위한 API
  - `그림 5-4` JTA를 통한 글로벌/분산 트랜잭션 관리
- 또 다른 문제 발생
  - 하이버네이트를 이용한 트랜잭션 관리 코드는 JDBC나 JTA의 코드와는 또 다르다.
    - Connection을 직접 사용하지 않고 Session을 사용하고, 독자적인 트랜잭션 관리 API를 사용한다.



#### 트랜잭션 API의 의존관계 문제와 해결책

- 여러 기술의 사용 방법에 공통점이 있다면 추상화를 생각해볼 수 있다.
  - 추상화란 하위 시스템의 공통점을 뽑아서 분리시키는 것을 말한다.
  - 하위 시스템이 어떤 것인지 알지 못해도, 또는 하위 시스템이 바뀌더라도 일관된 방법으로 접근할 수가 있다.
- 다양한 DB에서 제공하는 DB 클라이언트 라이브러리와 API는 서로 호환이 되지 않는 독자적인 방식으로 만들어져 있다.
  - 하지만 모두 SQL을 이용하는 방식이라는 공통점이 있다.   
  - 이 공통점을 뽑아내 추상화한 것이 JDBC다.
- 트랜잭션 처리 코드에도 추상화를 도입해볼 수 있지 않을까?
  - JDBC, JTA, 하이버네이트, JPA, JDO, 심지어는 JMS도 트랜잭션 개념을 갖고 있으니 모두 그 트랜잭션 경계설정 방법에서 공통점이 있을 것이다.
    - 이 공통적인 특징을 모아서 추상화된 트랜잭션 관리 계층을 만들 수 있다.
  - 애플리케이션 코드에서 트랜잭션 추상 계층이 제공하는 API를 이용해 트랜잭션을 이용하게 만들어준다면?
    - 특정 기술에 종속되지 않는 트랜잭션 경계설정 코드를 만들 수 있을 것이다.



#### 스프링의 트랜잭션 서비스 추상화

- `그림 5-6` 스프링의 트랜잭션 추상화 계층
  - 추상 인터페이스 PlatformTransactionManager
  - TransactionStatus 클래스

```java
public void upgradeLevels() {
  	PlatformTransactionManager transactionManager =
      	new DataSourceTransactionManager(dataSource);
		TransactionStatus status = 
      	transactionManager.getTransaction(new DefaultTransactionDefinition());
		try {
        List<User> users = userDao.getAll();
        for (User user : users) {
            if (canUpgradeLevel(user)) {
              upgradeLevel(user);
            }
        }
        this.transactionManager.commit(status);
		} catch (RuntimeException e) {
        this.transactionManager.rollback(status);
        throw e;
		}
}
```



#### 트랜잭션 기술 설정의 분리

- 트랜잭션 추상화 API를 적용한 UserService 코드
  - JTA를 이용하는 글로벌 트랜잭션으로 변경
    
    - ```java
      PlatformTransactionManager txManager = new JTATransactionManager();
      ```
  - 하이버네이트로 UserDao를 구현했다면 HibernateTransactionManager를 사용
  - JPA를 적용했다면 JPATransactionManager를 사용
  - 어떤 트랜잭션 매니저 구현 클래스를 사용할지 UserService 코드가 알고 있는 것은 DI 원칙에 위배
    
    - 자신이 사용할 구체적인 클래스를 스스로 결정하고 생성하지말고 컨테이너를 통해 외부에서 제공받게 하는 스프링 DI의 방식으로 변경
- 어떤 클래스든 스프링의 빈으로 등록할 때 먼저 검토 해야 할 것
  - 싱글톤으로 만들어져 여러 스레드에서 동시에 사용해도 괜찮은가
    - 상태를 갖고 있고, 멀티스레드 환경에서 안전하지 않은 클래스를 빈으로 무작정 등록하면 심각한 문제가 발생하기 때문이다.
  - 스프링이 제공하는 모든 PlatformTransactionManager의 구현 클래스는 싱글톤으로 사용이 가능하다.



## 5.3 서비스 추상화와 단일 책임 원칙



#### 수직, 수평 계층구조와 의존관계

- `그림 5-7` 계층과 책임의 분리
  - 애플리케이션의 비즈니스 로직과 그 하위에서 동작하는 로우레벨의 트랜잭션 기술이라는 아예 다른 계층의 특성을 갖는 코드를 분리

- DI의 가치는 관심, 책임, 성격이 다른 코드를 깔끔하게 분리하는데 있다.
  - 애플리케이션 로직의 종류에 따른 수평적인 구분
  - 로직과 기술이라는 수직적인 구분
  - 둘 모두 결합도가 낮으며, 서로 영향을 주지 않고 자유롭게 확장될 수 있는 구조를 만들 수 있는 데는 스프링의 DI가 중요한 역할을 하고 있다.



#### 단일 책임 원칙



#### 단일 책임 원칙의 장점

- 스프링의 의존관계 주입 기술인 DI는 모든 스프링 기술의 기반이 되는 핵심 엔진이자 원리이며, 스프링이 지지하고 지원하는, 좋은 설계와 코드를 만드는 모든 과정에서 사용되는 가장 중요한 도구다.
  - DAO와 DataSource를 분리할 때도 DI를 이용
  - 효과적인 단위 테스트를 만드는 과정에도 DI를 적용
  - 템플릿/콜백 패턴도 역시 DI를 응용

- DI의 원리를 잘 활용해서 스프링을 열심히 사용하다 보면, 어느 날 자신이 만든 코드에 객체지향 원칙과 디자인 패턴의 장점이 잘 녹아 있다는 사실을 발견하게 될 것이다.
  - 그것이 스프링을 사용함으로써 얻을 수 있는 가장 큰 장점이다.



## 5.4 메일 서비스 추상화

- 레벨이 업그레이드되는 사용자에게는 안내 메일을 발송해달라는 새로운 요청사항



### 5.4.1 JavaMail을 이용한 메일 발송 기능



#### JavaMail 메일 발송



### 5.4.2 JavaMail이 포함된 코드의 테스트



### 5.4.3 테스트를 위한 서비스 추상화

- 실제 메일 전송을 수행하는 JavaMail 대신에 테스트에서 사용할, JavaMail과 같은 인터페이스를 갖는 오브젝트를 만들어서 사용하면 문제는 모두 해결된다.



#### JavaMail을 이용한 테스트의 문제점

- JavaMail에는 구현을 바꿔치기할 만한 인터페이스가 보이지 않는다.
- JavaMail처럼 테스트하기 힘든 구조인 API를 테스트하기 좋게 만드는 방법이 있다.
  - 서비스 추상화를 적용



#### 메일 발송 기능 추상화

- JavaMail의 서비스 추상화 인터페이스 MailSender
- JavaMail을 사용해 메일 발송 기능을 제공하는 JavaMailSenderImpl 클래스
- 이전에는 JavaMail API를 직접 사용해서 메일 발송
  - 추상화로 한겹 감싸서 구체적인 기술이 보이지 않게한 MailSender 구현체 사용
  - try/catch 블록 등이 사라지면서 코드가 깔끔해진다.
  - MailSender 인터페이스 타입의 테스트용 구현체를 만들어서 테스트에서 사용할 수 있다.



#### 테스트용 메일 발송 오브젝트



#### 테스트와 서비스 추상화



### 5.4.4 테스트 대역



#### 의존 오브젝트의 변경을 통한 테스트 방법

- 하나의 오브젝트가 사용하는 오브젝트를 DI에서 의존 오브젝트라고 불러왔다.
  - 의존한다는 말은 종속되거나 기능을 사용한다는 의미다.
  - 작은 기능이라도 다른 오브젝트의 기능을 사용하면, 사용하는 오브젝트의 기능이 바뀌었을 때 자신이 영향을 받을 수 있기 때문에 의존하고 있다고 말한다.

- 트랜잭션과 메일의 추상화 과정에서도 살펴봤듯이 실전에서 사용할 오브젝트를 교체하지 않더라도, 단지 테스트만을 위해서도 DI는 유용하다.



#### 테스트 대역의 종류와 특징

- 테스트 환경을 만들어주기 위해, 테스트 대상이 되는 오브젝트의 기능에만 충실하게 수행하면서 빠르게, 자주 테스트를 실행할 수 있도록 사용하는 이런 오브젝트를 통틀어서 테스트 대역(test double)이라고 부른다.
- 대표적인 테스트 대역은 테스트 스텁(test stub)이다.
  - 테스트 스텁은 테스트 대상 오브젝트의 의존객체로서 존재하면서 테스트 동안에 코드가 정상적으로 수행할 수 있도록 돕는 것을 말한다.
  - 일반적으로 테스트 스텁은 메소드를 통해 전달하는 파라미터와 달리, 테스트 코드 내부에서 간접적으로 사용된다.
    - 따라서 DI 등을 통해 미리 의존 오브젝트를 테스트 스텁으로 변경해야 한다.



#### 목 오브젝트를 이용한 테스트

- 테스트가 수행될 수 있도록 의존 오브젝트에 간접적으로 입력 값을 제공해주는 스텁 오브젝트
- 간접적인 출력 값까지 확인이 가능한 목 오브젝트
- 두 가지는 테스트 대역의 가장 대표적인 방법이며 효과적인 테스트 코드를 작성하는 데 빠질 수 없는 중요한 도구다.



## 5.5 정리

- 비즈니스 로직을 담은 코드는 데이터 액세스 로직을 담은 코드와 깔끔하게 분리되는 것이 바람직하다.
  - 비즈니스 로직 코드 또한 내부적으로 책임과 역할에 따라서 깔끔하게 메소드로 정리돼야 한다.
- 트랜잭션 경계설정 코드가 비즈니스 로직 코드에 영향을 주지 않게 하려면 스프링이 제공하는 트랜잭션 서비스 추상화를 이용하면 된다.
- 서비스 추상화는 로우레벨의 트랜잭션 기술과 API의 변화에 상광없이 일관된 API를 가진 추상화 계층을 도입한다.
- 서비스 추상화는 테스트하기 어려운 JavaMail 같은 기술에도 적용할 수 있다.
  - 테스트를 편리하게 작성하도록 도와주는 것만으로도 서비스 추상화는 가치가 있다.

