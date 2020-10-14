

## 3.1 다시 보는 초난감 DAO



### 3.1.1 예외처리 기능을 갖춘 DAO



#### JDBC 수정 기능의 예외처리 코드

- Connection이나 PrepareStatement의 close() 호출을 통해 가져온 리소스를 반환
  - close() 호출하기 전에 null 체크 해줘야 한다.
  - close() 메소드도 SQLException이 발생할 수 있는 메소드다.
    - try/catch 문으로 잡아줘야 한다.



#### JDBC 조회 기능의 예외처리



## 3.2 변하는 것과 변하지 않는 것



### 3.2.1 JDBC try/catch/finally 코드의 문제점

- 문제의 핵심은 변하지 않는, 그러나 많은 곳에서 중복되는 코드와 로직에 따라 자꾸 확장되고 자주 변하는 코드를 잘 분리해내는 작업
  - DAO와 DB 연결 기능을 분리하는 것과는 성격이 다르기 때문에 해결 방법이 조금 다르다.



### 3.2.2 분리와 재사용을 위한 디자인 패턴 적용



#### 메소드 추출



#### 템플릿 메소드 패턴의 적용

- 템플릿 메소드 패턴은 상속을 통해 기능을 확장해서 사용
  - 이 경우에는 상속을 통해 확장을 꾀하는 템플릿 메소드 패턴의 단점이 고스란히 드러난다.



#### 전략 패턴의 적용

- 전략 패턴은 필요에 따라 컨텍스트는 그대로 유지되면서(OCP의 폐쇄 원칙) 전략을 바꿔 쓸 수 있다(OCP의 개방 원칙).
  - 그런데 이렇게 컨텍스트 안에서 이미 구체적인 전략 클래스인 DeleteAllStatement를 사용하도록 고정되어 있다면 뭔가 이상하다.



#### DI 적용을 위한 클라이언트/컨텍스트 분리



## 3.3 JDBC 전략 패턴의 최적화



### 3.3.1 전략 클래스의 추가 정보

- deleteAll() 과는 달리 add()에서는 PreparedStatement를 만들 때 User라는 부가적인 정보가 필요하다.
  - StatementStrategy를 구현하는 AddStatement를 만들고 생성자를 통해 User를 제공 



### 3.3.2 전략과 클라이언트의 동거



#### 로컬 클래스



#### 익명 내부 클래스



## 3.4 컨텍스트와 DI



### 3.4.1 JdbcContext의 분리



#### 클래스 분리

- workWith~ 메소드 명
  - StatementStrategy를 인자로 받는 workWithStatementStrategy

- JdbcContext 클래스로 분리해서 Dao에 DI



#### 빈 의존관계 변경

- `그림 3-5` JdbcContext가 적용된 빈 오브젝트 관계 참고



### 3.4.2 JdbcContext의 특별한 DI



#### 스프링 빈으로 DI

- 인터페이스 사용 여부
  - 왜 인터페이스를 사용하지 않았을까?
  - 인터페이스가 없다는 건 매우 긴밀한 관계를 가지고 강하게 결합되어 있다는 의미
    - UserDao는 항상 JdbcContext 클래스와 함께 사용돼야 한다.
      - 강한 응집도
    - UserDao가 JDBC 방식 대신 JPA나 하이버네이트 같은 ORM을 사용해야 한다면 JdbcContext도 통째로 바뀌어야 한다.



#### 코드를 이용하는 수동 DI

- `리스트 3-25` JdbcContext 생성과 DI 작업을 수행하는 setDataSource() 메소드 참고
- 두가지 방식의 장단점
  - 관계가 외부에 들어나고 안나고



## 3.5 템플릿과 콜백

- 전략 패턴의 기본 구조에 익명 내부 클래스를 활용한 방식
  - 이런 방식을 스프링에서는 템플릿/콜백 패턴이라고 부른다.
- 템플릿
  - 템플릿은 어떤 목적을 위해 미리 만들어둔 모양이 있는 틀
  - 템플릿 메소드 패턴은 고정된 틀의 로직을 가진 템플릿 메소드를 슈퍼클래스에 두고, 바뀌는 부분을 서브클래스의 메소드에 두는 구조
- 콜백
  - 파라미터로 전달되지만 값을 참조하기 위한 것이 아니라 특정 로직을 담은 메소드를 실행시키기 위해 사용한다.
  - 자바에서는 메소드 자체를 파라미터로 전달할 방법은 없기 때문에 메소드가 담긴 오브젝트를 전달해야 한다.
  - functional object라고도 한다.
- 콜백이 보이면 템플릿 등에서 호출되는 시점을 생각하면 덜 복잡해 보인다.
  - 콜백을 호출하는 쪽에서는 `XxxCallback xxx` 콜백이 파라미터로 들어오면 `xxx.doSomething(Y y)`
    - 콜백을 만들 때 인자값은 콜백을 사용하는 쪽에서 파라미터로 넣어준다.
  - 262p 콜백이 두개인 것도 이렇게 보면 쉽다.



### 3.5.1 템플릿/콜백의 동작원리



#### 템플릿/콜백의 특징



#### JdbcContext에 적용된 템플릿/콜백



### 3.5.2 편리한 콜백의 재활용



#### 콜백의 분리와 재활용

- `리스트 3-27` 변하지 않는 부분을 분리시킨 deleteAll() 메소드 참고
  - 익명 내부 클래스를 또 변하는 부분은 인자로 받는 메소드로 분리했다.



#### 콜백과 템플릿의 결합

- 메소드로 분리 시킨 익명 내부 클래스의 변하지 않는 부분을 템플릿 클래스로 이동



### 3.5.3 템플릿/콜백의 응용

- 템플릿/콜백 패턴 연습 해보기 좋은 간단한 예제



#### 테스트와 try/catch/finally



#### 중복의 제거와 템플릿/콜백 설계

- 템플릿/콜백 패턴을 적용할 때는 템플릿과 콜백의 경계를 정하고 템플릿이 콜백에게, 콜백이 템플릿에게 각각 전달하는 내용이 무엇인지 파악하는게 가장 중요하다.
  - 그에 따라 콜백의 인터페이스를 정의해야 하기 때문

```java
public Integer calcSum(String filepath) throws IOException {
  	BufferedReaderCallback sumCallback =
    		new BufferedReaderCallback() {
      			public Integer doSomethingWithReader(BufferedReader br) throws IOException {
              	Integer sum = 0;
              	String line = null;
              	while((line = br.readLine()) != null) {
                  	sum += Integer.valueOf(line);
                }
              	return sum;
            }
    		};
  	return fileReadTemplate(filepath, sumCallback);
}
```

```java
public Integer calcMultiply(String filepath) throws IOException {
  	BufferedReaderCallback multiplyCallback =
    		new BufferedReaderCallback() {
      			public Integer doSomethingWithReader(BufferedReader br) throws IOException {
              	Integer multiply = 1;
              	String line = null;
              	while((line = br.readLine()) != null) {
                  	multiply *= Integer.valueOf(line);
                }
              	return multiply;
            }
    		};
  	return fileReadTemplate(filepath, multiplyCallback);
}
```



#### 템플릿/콜백의 재설계

```java
public Integer doSomethingWithReader(BufferedReader br) throws IOException {
    Integer sum = 0;
    String line = null;
    while((line = br.readLine()) != null) {
      	sum += Integer.valueOf(line);
    }
    return sum;
}
```

```java
public Integer doSomethingWithReader(BufferedReader br) throws IOException {
    Integer multiply = 1;
    String line = null;
    while((line = br.readLine()) != null) {
     	 multiply *= Integer.valueOf(line);
    }
    return multiply;
}
```

- 위 코드를 보면 공통적인 패턴이 발견된다.
  - `sum = 0`, `multiply = 1`
    - 이 부분은 템플릿을 호출하는 부분에서 파라미터로 넣게 리팩토링 
  - `+=`, `*=`
- 공통적인 패턴을 콜백에서 템플릿으로 넘긴다.
  - 콜백과 템플릿 네이밍 변경

```java
public Integer calcSum(String filepath) throws IOException {
  	LineCallback sumCallback =
    		new LineCallback() {
      			public Integer doSomethingWithLine(String line, Integer value) throws IOException {
              	return value + Integer.valueOf(line);
            }
    		};
  	return lineReadTemplate(filepath, sumCallback, o);
}
```

```java
public Integer calcSum(String filepath) throws IOException {
  	LineCallback multiplyCallback =
    		new LineCallback() {
      			public Integer doSomethingWithLine(String line, Integer value) throws IOException {
              	return value * Integer.valueOf(line);
            }
    		};
  	return lineReadTemplate(filepath, multiplyCallback, 1);
}
```



#### 제네릭스를 이용한 콜백 인터페이스

```java
public interface LineCallback<T> {
  	T doSomethingWithLine(String line, T value);
}
```

```java
public <T> T lineReadTemplate(String filepath, LineCallback<T> callback, T initVal) throws IOException {
  	BufferedReader br = null;
  	try {
        T res = initVal;
        String line = null;
        while((line = br.readLine()) != null) {  
            res = callback.doSomethingWithLine(line, res);  
        }    
    		return res;  
    }
  	catch(IOException e) {...}
  	finally {...}
}
```



## 3.6 스프링의 JdbcTemplate

- 스프링은 JDBC를 이용하는 DAO에서 사용할 수 있도록 준비된 다양한 템플릿과 콜백을 제공
  - 거의 모든 종류의 JDBC 코드에 사용 가능한 템플릿과 콜백을 제공
  - 자주 사용되는 패턴을 가진 콜백은 다시 템플릿에 결합시켜서 간단한 메소드 호출만으로 사용이 가능



### 3.6.1 update()



### 3.6.2 queryForInt()



### 3.6.3 queryForObject()

- RowMapper 콜백
  - 템플릿으로부터 ResultSet을 전달받고, 필요한 정보를 추출해서 리턴하는 방식으로 동작
  - 이름 그대로 디비에서 가지고 온 데이터 로우들을 내가 원하는 타입으로 맵핑을 해서 반환



### 3.6.4 query()



#### 기능 정의와 테스트 작성



#### query() 템플릿을 이용하는 getAll() 구현



#### 테스트 보완



### 3.6.5 재사용 가능한 콜백의 분리



#### DI를 위한 코드 정리



#### 중복 제거



#### 템플릿/콜백 패턴과 UserDao



## 3.7 정리



#### 예제 3.3

**템플릿 메소드 패턴**(상속을 통한 확장)에서 **전략 패턴**으로

변하지 않는 부분을 메소드로 분리 `jdbcContextWithStatementStrategy()`

변하는 부분을 추상화한 인터페이스 타입(`StatementStrategy`)을 메소드의 파라미터로 넘긴다.

`jdbcContextWithStatementStrategy(StatementStrategy stmt)`

`add(final User user)` 메소드 안에서

`StatementStrategy` 재정의해서 `jdbcContextWithStatementStrategy()` 메소드의 파라미터로 

```java
public void add(final User user) throws SQLException {
		jdbcContextWithStatementStrategy(
        new StatementStrategy() {			
            public PreparedStatement makePreparedStatement(Connection c)
							...
            }
        }
		);
	}
```

```java
public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
			
		...
  
    c = dataSource.getConnection();
  	ps = stmt.makePreparedStatement(c);
		
		...
      
}
```



#### 예제 3.4

변하지 않는 부분 클래스로 분리 

`jdbcContextWithStatementStrategy()`  - > `JdbcContext` 



#### 예제 3.5.2

```java
public void executeSql(final String query) throws SQLException {
    workWithStatementStrategy(
        new StatementStrategy() {
            public PreparedStatement makePreparedStatement(Connection c)
                throws SQLException {
                return c.prepareStatement(query);
            }
        }
    );
}	
```

```java
public void deleteAll() throws SQLException {
		this.jdbcContext.executeSql("delete from users");
}
```



#### 예제 3.final

```java
private JdbcTemplate jdbcTemplate;

private RowMapper<User> userMapper = 
    new RowMapper<User>() {
        public User mapRow(ResultSet rs, int rowNum) throws SQLException {
          User user = new User();
          user.setId(rs.getString("id"));
          user.setName(rs.getString("name"));
          user.setPassword(rs.getString("password"));
          return user;
        }
		};

public void add(final User user) {
  	this.jdbcTemplate.update("insert into users(id, name, password) values(?,?,?)",
                           user.getId(), user.getName(), user.getPassword());
}
```




