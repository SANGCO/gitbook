- 지금까지 만든 DAO에 트랜잭션을 적용 하면서
  - 성격이 비슷한 여러 종류의 기술을 추상화
    - 일관된 방법으로 사용할 수 있도록 지원



## 5.1 사용자 레벨 관리 기능 추가



### 5.1.1 필드 추가



#### Level 이늄



#### User 필드 추가



#### UserDaoTest 테스트 수정



#### UserDaoJdbc 수정

- DB까지 연동되는 테스트를 잘 만들어 두먼 SQL 문장에 오타도 빠르게 잡아낼 수 있다.
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
- 업그레이드 순서를 담고 있도록 enum Level 수정
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



#### 트랜잭션 API의 의존관계 문제와 해결책



#### 스프링의 트랜잭션 서비스 추상화



#### 트랜잭션 기술 설정의 분리



#### 수직, 수평 계층구조와 의존관계



## 5.3 서비스 추상화와 단일 책임 원칙



#### 단일 책임 원칙

#### 단일 책임 원칙의 장점





## 5.4 메일 서비스 추상화



### 5.4.1 JavaMail을 이용한 메일 발송 기능



#### JavaMail 메일 발송



### 5.4.2 JavaMail이 포함된 코드의 테스트



### 5.4.3 테스트를 위한 서비스 추상화



#### JavaMail을 이용한 테스트의 문제점

#### 메일 발송 기능 추상화

#### 테스트용 메일 발송 오브젝트

#### 테스트와 서비스 추상화



### 5.4.4 테스트 대역



#### 의존 오브젝트의 변경을 통한 테스트 방법

#### 테스트 대역의 종류와 특징

#### 목 오브젝트를 이용한 테스트





## 5.5 정리



