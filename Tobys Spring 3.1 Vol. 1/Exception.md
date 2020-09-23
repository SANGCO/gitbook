

## 4.1 사라진 SQLException

- `JdbcTemplate` 적용 이전에는 있었던 `throws SQLException` 선언이 적용 후에는 사라졌다.
  - `SQLException`은 어디로 간 것일까?

```java
public void deleteAll() throws SQLException {
  	// jdbcTemplate 적용 전
  	this.jdbcContext.executeSql("delete from users");
}
```

```java
public void deleteAll() {		// throws SQLException 사라졌다.
  	// jdbcTemplate 적용 후
  	this.jdbcTemplate.update("delete from users");
}
```



### 4.1.1 초난감 예외처리



#### 예외 블랙홀

- 굳이 예외를 잡아서 뭔가 조치를 취할 방법이 없다면 잡지 말아야 한다.
  - 메소드에 `throws SQLException`을 선언해서 메소드 밖으로 던지고 자신을 호출한 코드에 예외처리 책임을 전가해보리자.

```java
// 리스트 4-1 초난감 예외처리 코드 1
try {
		...  
} catch(SQLException e) {  	
}
```

```java
// 리스트 4-2 초난감 예외처리 코드 2
} catch(SQLException e) { 
  	System.out.printLn(e);
}
```

```java
// 리스트 4-3 초난감 예외처리 코드 3
} catch(SQLException e) {  
  	e.printStackTrace();
}
```

```java
// 리스트 4-4 그나마 나은 예외처리
} catch(SQLException e) {  
  	e.printStackTrace();
  	System.exit(1);
}
```



#### 무의미하고 무책임한 throws

- catch 블록으로 예외를 잡아봐야 해결할 방법도 없고 JDK API나 라이브러리가 던지는 각종 이름도 긴 예외들을 처리하는 코드를 매번 throws로 선언하기도 귀찮아지기 시작하면 메소드 선언에 `throws Exception`을 기계적으로 붙이는 개발자도 있다.
  - 이런 메소드 선언에서는 의미 있는 정보를 얻을 수 없다.
  - 정말 무엇인가 실행 중에 예외적인 상황이 발생할 수 있다는 것인지, 아니면 그냥 습관적으로 복사해서 붙여놓은 것인지 알 수가 없다.

```java
// 리스트 4-5 초난감 예외처리 4
public void method1() throws Exception {
  	method2();
  	...
}

public void method2() throws Exception {
  	method3();
  	...
}

public void method3() throws Exception {
  	...
}
```



### 4.1.2 예외의 종류와 특징

- `Error`
- `Exception`과 체크 예외
- `RuntimeException`과 언체크/런타임 예외
  - 이런 예외는 코드에서 미리 조건을 체크 하도록 주의 깊게 만든다면 피할 수 있지만 개발자가 부주의해서 발생 할 수 있는 경우에 발생하도록 만든 것이 런타임 예외다. 



### 4.1.3 예외처리 방법



#### 예외 복구

- 예외로 인해 기본 작업 흐름이 불가능하면 다른 작업 흐름으로 자연스럽게 유도
  - 사용자가 요청한 파일을 읽으려고 시도했는데 해당 파일이 없다거나 다른 문제가 있어서 잃히지 않아서 `IOException` 발생했다고 생각해보자.
    - 사용자에게 상황을 알려주고 다른 파일을 이용하도록 안내해서 예외상황을 해결할 수 있다.
- 재시도
  - 네트워크 접속이 원할하지 않아서 예외가 발생했다면 일정 시간 대기했다가 다시 접속을 시도해보는 방법을 사용해서 예외상황으로부터 복구를 시도할 수 있다.

```java
int maxretry = MAX_RETRY;
while(maxretry-- > 0) {
  	try {
      	...							// 예외가 발생할 가능성이 있는 시도
        return;  				// 작업 성공
    } catch(SomeException e) {
      	// 로그 출력. 정해진 시간만큼 대기
    } finally {
      	// 리소스 반납. 정리 작업
    }  
}
throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외 발생
```



#### 예외처리 회피

```java
// 리스트 4-7 예외처리 회피 1
public void add() throws SQLException {
  	// JDBC API
}
```

```java
// 리스트 4-8 예외처리 회피 2
public void add() throws SQLException {
  	try {
  			// JDBC API
    } catch(SQLException e) {
      	// 로그 출력
      	throw e;
    }  
}
```



#### 예외 전환

- 대부분 서버환경에서는 애플리케이션 코드에서 처리하지 않고 전달된 예외들을 일괄적으로 다룰 수 있는 기능을 제공한다.
  - 어차피 복구하지 못할 예외라면 애플리케이션 코드에서는 런타임 예외로 포장해서 던져버리고, 예외처리 서비스 등을 이용해 자세한 로그를 남기고, 관리자에게는 메일 등으로 통보해주고, 사용자에게는 친절한 안내 메시지를 보여주는 식으로 처리하는 게 바람직하다.

```java
// 리스트 4-9 예외 전환 기능을 가진 DAO 메소드
public void add(User user) throws DuplicateUserIdException, SQLException {
  	try {
  			// JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
        // 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
    } catch(SQLException e) {
      	// ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환
      	if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
          	throw new DuplicateUserIdException();
      	else
      			throw e; // 그 외의 경우는 SQLException 그대로
    }  
}
```

```java
// 리스트 4-10 중첩 예외 1
} catch(SQLException e) { 
		...
    throw DuplicateUserIdException(e);
}
```

```java
// 리스트 4-10 중첩 예외 2
} catch(SQLException e) { 
  	...
    throw DuplicateUserIdException().initCause(e);
}
```

```java
// 리스트 4-12 예외 포장
try {
		OrderHome orderHome = EJBHomeFactory.getInstance().getOrderHome();
  	Order order = orderHome.findByPrimaryKey(Integer id);
  
} catch (NamingException ne) {
  	throw new EJBException(ne);
  
} catch (SQLException se) {
  	throw new EJBException(se);
  
} catch (RemoteException re) {
  	throw new EJBException(re);
}  
```



### 4.1.4 예외처리 전략



#### 런타임 예외의 보편화

- 자바 초기부터 있었던 JDK의 API와 달리 최근에 등장하는 표준 스펙 또는 오픈소스 프레임워크에서는 API가 발생시키는 예외를 체크 예외 대신 언체크 예외로 정의하는 것이 일반화되고 있다.
  - 예전에는 복구할 가능성이 조금이라도 있다면 체크 예외로 만든다고 생각했는데, 지금은 항상 복구할 수 있는 예외가 아니라면 일단 언체크 예외로 만드는 경향이 있다.
  - 언체크 예외라도 필요하다면 얼마든지 catch 블록으로 잡아서 복구하거나 처리할 수 있다.



#### add() 메소드의 예외처리

- 리스트 4-14 add() 메소드를 사용하는 오브젝트는 `SQLException`을 처리하기 위해 불필요한 `throws` 선언을 할 필요는 없으면서, 필요한 경우 아이디 중복 상황을 처리하기 위해 `DuplicatedUserIdException`을 이용할 수 있다.
- 런타임 예외를 일반화해서 사용하는 방법은 여러모로 장점이 많다.
  - 단, 런타임 예외로 만들었기 때문에 사용에 더 주의를 기울일 필요도 있다.
  - 컴파일러가 예외처리를 강제하지 않으므로 신경 쓰지 않으면 예외상황을 충분히 고려하지 않을 수도 있기 때문이다.
  - 런타임 예외를 사용하는 경우에는 API 문서나 레퍼런스 문서 등을 통해, 메소드를 사용할 때 발생할 수 있는 예외의 종류와 원인, 활용 방법을 자세히 설명해두자.

```java
// 리스트 4-14 예외처리 전략을 적용한 add()
public void add() throws DuplicateUserIdException {
  	try {
  			// JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
        // 그런 기능이 있는 다른 SQLException을 던지는 메소드를 호출하는 코드
    } catch(SQLException e) {
      	if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
          	throw DuplicateUserIdException();			// 예외 전환
      	else
      			throw new RuntimeException(e);				// 예외 포장 
    }  
}
```



#### 애플리케이션 예외

- 애플리케이션 예외
  - 시스템 또는 외부의 예외상황이 원인이 아니라 애플리케이션 자체의 로직에 의해 의도적으로 발생시키고, 반드시 catch 해서 무엇인가 조치를 취하도록 요구하는 예외
- 사용자가 요청한 금액을 은행계좌에서 출금하는 기능을 가진 메소드
  - 현재 잔고를 확인하고, 허용하는 범위를 넘어서 출금을 요청하면 출금 작업을 중단시키고, 적절한 경고를 사용자에게 보내야 한다.
  - 이런 기능을 담은 메소드를 설계하는 두 가지 방법
    - 정상적인 출금처리를 했을 경우와 잔고 부족이 발생했을 경우에 각각 다른 종류의 리턴 값을 돌려준다.
      - 이렇게 리턴 값으로 결과를 확인하고, 예외상황을 체크하면 불편한 점도 있다.
        - 예외상황에 대한 리턴 값을 명확하게 코드화하고 잘 관리하지 않으면 혼란이 생길 수 있다.
        - 결과 값을 확인하는 조건문이 자주 등장해서 코드는 지저분해지고 흐름을 파악하고 이해하기가 힘들어진다.
    - 정상적인 흐름을 따르는 코드는 그대로 두고, 잔고 부족과 같은 예외 상황에서는 비즈니스적인 의미를 띤 예외를 던진다.
      - 이때 사용하는 예외는 의도적으로 체크 예외로 만들어 개발자가 잊지 않고 잔고 부족처럼 자주 발생 가능한 예외상황에 대한 로직을 구현하도록 강제해주는 게 좋다.

```java
// 리스트 4-15 애플리케이션 예외를 사용한 코드
try {
		BigDecimal balance = account.withdraw(amount);
  	...
    // 정상적인 처리 결과를 출력하도록 진행
} catch(InsufficientBalanceException e) {			// 체크 예외
  	// InsufficientBalanceException에 담긴 인출 가능한 잔고금액 정보를 가져옴
  	BigDecimal availFunds = e.getAvailFunds();
  	...
    // 잔고 부족 안내 메시지를 준비하고 이를 출력하도록 진행  
}  
```



### 4.1.5 SQLException은 어떻게 됐나?

- 지금까지 다룬 예외처리에 대한 내용은 JdbcTemplate을 적용하는 중에 throws SQLException 선언이 왜 사라졌는가를 설명하는 데 필요한 것이었다.
  - 스프링의 예외처리 전략과 원칙을 알고 있어야 하기 때문이다.
- 스프링의 JdbcTemplate은 언체크/런타임 예외로 전환하는 예외처리 전략을 따르고 있다.
  - JdbcTemplate 템플릿과 콜백 안에서 발생하는 모든 SQLException을 런타임 예외인 DataAccessException으로 포장해서 던져준다.
  - 따라서 JdbcTemplate을 사용하는 UserDao 메소드에선 꼭 필요한 경우에만 런타임 예외인 DataAccessException을 잡아서 처리하면 되고 그 외의 경우에는 무시해도 된다.
    - 그래서 DAO 메소드에서 SQLException이 모두 사라진 것이다.
- 스프링의 API 메소드에 정의되어 있는 대부분의 예외는 런타임 예외다.
  - 따라서 발생 가능한 예외가 있다고 하더라도 이를 처리하도록 강제하지 않는다.



## 4.2 예외 전환

- 에외를 다른 것으로 바꿔서 던지는 예외 전환의 목적은 두 가지
  - 런타임 예외로 표장해서 굳이 필요하지 않은 catch/throws를 줄이기
  - 로우레벨의 예외를 좀 더 의미 있고 추상화된 예외로 바꿔서 던져주기



### 4.2.1 JDBC의 한계



#### 비표준 SQL



#### 호환성 없는 SQLException의 DB 에러정보

- DB마다 SQL만 다른 것이 아니라 에러의 종류와 원인도 제각각이다.
  - 그래서 JDBC는 데이터 처리 중에 발생하는 다양한 예외를 그냥 `SQLException` 하나에 모두 담아버린다.
    - JDBC API는 이 `SQLException` 한 가지만 던지도록 설계되어 있다.

- 호환성 없는 에러 코드와 표준을 잘 따르지 않는 상태 코드를 가진 `SQLException`만으로 DB에 독립적인 유연한 코드를 작성하는 건 불가능에 가깝다.



### 4.2.2 DB 에러 코드 매핑을 통한 전환



### 4.2.3 DAO 인터페이스와 DataAccessException 계층구조

- 스프링이 왜 이렇게 `DataAccessException` 계층구조를 이용해 기술에 독립적인 예외를 정의하고 사용하게 하는지 생각해보자.



#### DAO 인터페이스와 구현의 분리

```java
// 리스트 4-19 기술에 독립적인 이상적인 DAO 인터페이스
public interface UserDao {
  	public void add(User user); // 이렇게 선언하는 것이 과연 가능할까?
}
```

- DAO에서 사용하는 데이터 액세스 기술의 API가 예외를 던지기 때문에 리스트 4-19의 메소드 선언은 사용할 수 없다.
  - UserDao를 구현하는 클래스가 JDBC API를 사용하면 add() 메소드는 `SQLException`을 던질 것이다.
    - 인터페이스의 메소드 선언에는 없는 예외를 구현 클래스 메소드의 `throws`에 넣을 수는 없다.
    - 따라서 인터페이스 메소드도 다음과 같이 선언돼야 한다.
      - `public void add(User user) throws SQLException;` 



```java
public interface UserDao {
    public void add(User user) throws SQLException;        // JDBC
    public void add(User user) throws PersistentException; // JPA
    public void add(User user) throws HibernateException;  // Hibernate
    public void add(User user) throws JdoException;				 // JDO
		...
}
```

- 데이터 액세스 기술의 API는 자신만의 독자적인 예외를 던지기 때문에 `SQLException`을 던지도록 선언한 인터페이스 메소드는 사용할 수 없다.
- 인터페이스로 메소드의 구현은 추상화했지만 구현 기술마다 던지는 예외가 다르기 때문에 메소드의 선언이 달라진다는 문제가 발생한다.
  - DAO 인터페이스를 기술에 완전히 독립적으로 만들려면 예외가 일치하지 않는 문제도 해결해야 한다.
    - `public void add(User user) throws Exception;`
      - 간단하긴 하지만 무책임한 선언

- 다행히 JDBC보다는 늦게 등장한 JDO, Hibernate, JPA 등의 기술은 런타임 예외를 사용한다. 
  - 따라서 JDBC를 이용한 DAO에서 모든 `SQLException`을 런타일 예외로 포장해주기만 하면 메소드 선언을 처음에 의도했던 대로 만들 수 있다.
    - `public void add(User user);`
      - `throw SQLException` 없이
      - 이제 DAO에서 사용하는 기술에 완전히 독립적인 인터페이스 선언이 가능해졌다.
        - 하지만 이것만으로 충분할까?

- 데이터 액세스 기술이 달라지면 같은 상황에서도 다른 종류의 예외가 던져진다.
  - DAO를 사용하는 클라이어트 입장에서는 DAO의 사용 기술에 따라서 예외 처리 방법이 달라져야 한다.
  - 결국 클라이언트가 DAO의 기술에 의존적이 될 수밖에 없다.
  - 단지 인터페이스로 추상화하고, 일부 기술에서 발생하는 체크 예외를 런타임 예외로 전환하는 것만으론 불충분하다.



#### 데이터 액세스 예외 추상화와 DataAccessException 계층구조

- 스프링은 자바의 다양한 액세스 기술을 사용할 때 발생하는 예외들을 추상화해서 `DataAccessException` 계층구조 안에 정리해놓았다.
- `JdbcTemplate`과 같이 스프링의 데이터 액세스 지원 기술을 이용해 DAO를 만들면 사용 기술에 독립적인 일관성 있는 예외를 던질 수 있다.
  - 결국 인터페이스 사용, 런타임 예외 전환과 함께 `DataAccessException` 예외 추상화를 적용하면 데이터 액세스 기술과 구현 방법에 독립적인 이상적인 DAO를 만들 수가 있다.



### 4.2.4 기술에 독립적인 UserDao 만들기



#### 인터페이스 적용



#### 테스트 보완



#### DataAccessException 활용 시 주의사항

- 데이터 액세스 기술을 하이버네이트나 JPA를 사용했을 때도 동일한 예외가 발생할 것으로 기대하지만 실제로 다른 예외가 던져진다.
  - 그 이유는 `SQLException`에 담긴 DB의 에러 코드를 바로 해석하는 JDBC의 경우와 달리 JPA나 하이버네이트, JDO 등에서는 각 기술이 재정의한 예외를 가져와 스프링이 최종적으로 `DataAccessException`으로 변환하는데, DB의 에러 코드와 달리 이런 예외들은 세분화되어 있지 않기 때문이다.
- `DataAccessException`이 기술에 상관없이 어느 정도 추상화된 공통 예외로 변환해주긴 하지만 근본적인 한계 때문에 완벽하다고 기대할 수는 없다. 
  - 사용에 주의를 기울여야 한다.
  - `DataAccessException`을 잡아서 처리하는 코드를 만들려고 한다면 미리 학습 테스트를 만들어서 실제로 전환되는 예외의 종류를 확인해둘 필요가 있다.



## 4.3 정리

- `SQLException`의 에러 코드는 DB에 종속되기 때문에 DB에 독립적인 예외로 전환될 필요가 있다.
- 스프링은 `DataAccessException`을 통해 DB에 독립적으로 적용 가능한 추상화된 런타임 예외 계층을 제공한다.
- DAO를 데이터 액세스 기술에서 독립시키려면 인터페이스 도입과 런타임 예외 전환, 기술에 독립적인 추상화된 예외로 전환이 필요하다.