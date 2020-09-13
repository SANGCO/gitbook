# 1장 오브젝트와 의존관계



## 1.1 초난감 DAO



### 1.1.1 User



### 1.1.2 UserDao

- 대략 남감한 코드
  - 리팩토링을 해야 할 포인트가 많이 보인다.

```java
public class UserDao {
  
    public void add(User user) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager
          .getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");

        PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }


    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager
          .getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
        PreparedStatement ps = c.prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }

}
```



### 1.1.3 main()을 이용한 DAO 테스트 코드



## 1.2 DAO의 분리



### 1.2.1 관심사의 분리

- 관심사 분리
  - 관심사가 같은거 끼리 모으고 다른 것은 분리해서 같은 관심에 효과적으로 집중
  - 변경이 일어났을 때 해당 관심사를 모아놓은 부분만 고칠 수 있도록



### 1.2.2 커넥션 만들기의 추출



#### UserDao의 관심사항

- DB와 연결을 위한 커넥션을 어떻게 가져올까
  - 현재 모든 메서드에 DB 커넥션을 가지고 오는 코드가 있다.
    - DB 커넥션 관련 변경이 일어나면 골때린다. 
- 사용자 등록을 위해 DB에 보낼 SQL 문장을 담을 Statement를 만들고 실행하는 것
- 작업이 끝나면 사용한 리소스인 Statement와 Connection 오브젝트를 닫아줘서 소중한 공유 리소를 시스템에 돌려주는 것



#### 중복 코드의 메소드 추출

```java
private Connection getConnection() throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager
      .getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
    return c;
}
```



#### 변경사항에 대한 검증: 리팩토링과 테스트



### 1.2.3 DB 커넥션 만들기의 독립



#### 상속을 통한 확장

- UserDao는 DB Connection 그 자체에만 관심이 있다.
  - 어떤 방식으로 Connection 오브젝트를 만들고 제공하는지는 NUserDao와 DUserDao의 관심사항임.
- 템플릿 메소드 패턴
  - 기본적인 로직은 UserDao에 있다.
    - 커넥션 가져오기, SQL 생성, 실행 ,반환
  - 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 필요에 맞게 구현
    - NUserDao, DUserDao에서 자신에 DB에 맞는 Connection을 생성해서 반환
- 팩토리 메소드 패턴
  - NUserDao, DUserDao가 Connection오브젝트를 생성하는 방식이 다르다.
  - NUserDao, DUserDao는 Connection 타입의 다른 구현체를 반환할 수 있다.

```java
public abstract class UserDao {
 
    ...

    abstract protected Connection getConnection() throws ClassNotFoundException, SQLException ;

    ...  
  
}  

```

```java
public class NUserDao extends UserDao {
    
  	protected Connection getConnection() throws ClassNotFoundException,SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager
          .getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
        return c;
    }
  
}
```



## 1.3 DAO의 확장





### 1.3.1 클래스의 분리

- 관심사 분리 작업을 점진적으로 진행해왔다.
  - 처음에는 독립된 메소드를 만들어서 분리
  - 다음에는 상하위 클래스로 분리
  - 이번에는 상속관계도 아닌 **완전히 독립적인 클래스**로 분리
    - 분리를 했는데도 특정 코드에 종속적이고 납품이 힘들었던 처음의 문제에 다시 봉착했네?

```java
public abstract class UserDao {
		
  	private SimpleConnectionMaker simpleConnectionMaker;
  
  	...
    
}    
```

```java
public class SimpleConnectionMaker {
    
  	public Connection getConnection() throws ClassNotFoundException,SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager
          .getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
        return c;
    }
  	
}
```



### 1.3.2 인터페이스의 도입

- 추상화
  - 두 개의 클래스가 서로 긴밀하게 연결되어 있지 않도록 중간에 추상적인 느슨한 연결고리를 만들어주기
  - 추상화란 어떤 것들의 공통적인 성격을 뽑아내어 이를 따로 분리해내는 작업
  - 자바가 추상화를 위해 제공하는 가장 유용한 도구는 인터페이스

```java
public interface ConnectionMaker {

		public abstract Connection makeConnection() throws ClassNotFoundException, SQLException;

}
```

```java
public class DConnectionMaker implements ConnectionMaker {
   
  	public Connection makeConnection() throws ClassNotFoundException, SQLException {
      	Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager
            .getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
        return c;
    }
  	
}
```

```java
public class UserDao {
	
    private ConnectionMaker connectionMaker;

    public UserDao() {
				// 여기서 또 문제 발생.
        // N사에 납품할 때는 우짤긴데?
      	this.connectionMaker = new DConnectionMaker();
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
				// makeConnection()이란 ConnectionMaker 인터페이스에 정의된 메소드를 사용해서 
        // 클래스가 바뀐다고 해도 따로 수정하지 않아도 된다.
      	Connection c = this.connectionMaker.makeConnection();

        ...

    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = this.connectionMaker.makeConnection();

        ...

    }

}
```



### 1.3.3 관계설정 책임의 분리

- ConnectionMaker의 관심사
  - DB 커넥션을 어떻게 가져올 것인가
- UserDao의 관심사
  - JDBC API와 User 오브젝트를 이용해 DB에 정보를 넣고 빼는 것
  - UserDao가 어떤 ConnectionMaker 구현 클래스의 오브젝트를 이용하게 할지를 결정하는 것
    - 이 관심사를 **UserDao에서 분리**시켜야 UserDao는 독립적으로 확장 가능한 클래스가 될 수 있다.
    - 이 관심사를 클라이언트에게 넘겨보자.
      - 클라이언트가 자신이 원하는 ConnectionMaker의 구현체를 만들어 UserDao에게 전달

```java
public UserDao(ConnectionMaker connectionMaker) {
 	 this.connectionMaker = connectionMaker;
}
```

```java
public class UserDaoTest {
   
  	public static void main(String[] args) throws ClassNotFoundException, SQLException {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao dao = new UserDao(connectionMaker);

       	...
          
    }
  	
}
```



### 1.3.4 원칙과 패턴



#### 개방 폐쇄 원칙

- UserDao는 DB 연결 방법이라는 기능을 **확장하는 데는 열려** 있다.
  - UserDao에 전혀 영향을 주지 않고도 얼마든지 기능을 확장할 수 있게 되어 있다.
- 동시에 UserDao 자신의 핵심 기능을 구현한 코드는 그런 변화에 영향을 받지 않고 유지할 수 있으므로
  **변경에는 닫혀** 있다고 말할 수 있다.



#### 높은 응집도와 낮은 결합도

- **책임, 관심사는 한곳으로 응집**되어 있어야한다.
- 대부분의 경우 결합도는 낮아야 좋다.
  - 자유로운 확장 변경이 가능하려면 **부품을 자유롭게 바꿔 끼울 수 있어야**한다. 



#### 전략 패턴

- 변경이 필요한 부분을 통째로 인터페이스를 통해 외부로 분리 시키고, 필요에 따라 구체적인 구현 클래스를 바꾸는 패턴
  - 개방 폐쇄 원칙의 실현에도 가장 잘 들어 맞는 패턴



## 1.4 제어의 역전(IoC)



### 1.4.1 오브젝트 팩토리

- UserDaoTest는 UserDao의 기능이 잘 동작하는지에 관심이 있다.
  - 다른 책임이나 관심사는 분리하자.
    - UserDao와 ConnectionMaker 구현 클래스의 오브젝트를 만드는 것
    - 생성된 두 개의 오브젝트를 서로 연결시켜주는 것



#### 팩토리

- 객체의 생성 방법을 결정하고 그렇게 만들어진 오브젝트를 돌려주는 일을 하는 오브젝트를 팩토리라고 부르다.
  - 디자인 패턴에 추상 팩토리 패턴이나 팩토리 메소드 패턴과는 다르다.
  - 단순히 오브젝트를 생성하는 쪽과 생성된 오브젝트를 사용하는 쪽의 역할과 책임을 분리하려는 목적으로 사용

```java
public class UserDaoFactory {
 
  	public UserDao userDao() {
      	ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao dao = new UserDao(connectionMaker());
        return dao;
    }

}
```

```java
public class UserDaoTest {
  
  	public static void main(String[] args) throws ClassNotFoundException, SQLException {
      	UserDao dao = new UserDaoFactory().userDao();

     		...
          
    }
  	
}
```



#### 설계도로서의 팩토리

- 그림 1-8 참고



### 1.4.2 오브젝트 팩토리의 활용

- DaoFactory에 UserDao가 아닌 다른 DAO 생성 기능을 넣기

```java
public class DaoFactory {

    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }
  
    public UserDao userDao() {
        return new AccountDao(connectionMaker());
    }
  
    public UserDao userDao() {
        return new MessageDao(connectionMaker());
    }

    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```



### 1.4.3 제어권의 이전을 통한 제어관계 역전

- **라이브러리**를 사용하는 애플리케이션 코드는 애플리케이션 흐름을 직접 제어한다.
- 반면에 **프레임워크**는 거꾸로 애플리케이션 코드가 프레임워크에 의해 사용된다.
  - 보통 프레임워크 위에 개발한 클래스를 등록해두고, 프레임워크가 흐름을 주도하는 중에 개발자가 만든 애플리케이션 코드를 사용하도록 만드는 방식



## 1.5 스프링의 IoC



### 1.5.1 오브젝트 팩토리를 이용한 스프링 IoC



#### 애플리케이션 컨텍스트와 설정정보

- 스프링에서는 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트를 **빈**이라고 부른다.
  - 동시에 스프링 빈은 컨테이너가 생성과 관계설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트를 가리키는 말이다.

- 스프링에서는 빈의 생성과 관계설정 같은 제어를 담당하는 IoC 오브젝트를 **빈 팩토리**라고 부른다.
  - 보통 빈 팩토리보다는 이를 좀 더 확장한 **애플리케이션 컨텍스트**를 주로 사용한다.
    - 애플리케이션 컨택스트는 IOC 방식을 따라 만들어진 일종의 빈 팩토리라고 생각하면 된다.
    - 빈 팩토리라고 말할 때는 **빈을 생성하고 관계를 설정하는 IoC의 기본 기능**에 초점을 맞춘 것
    - 애플리케이션 컨텍스트라고 말할 때는 **애플리케이션 전반에 걸쳐 모든 구성요소의 제어 작업을 담당하는 IoC 엔진**이라는 의미가 좀 더 부각된다고 보면 된다.



#### DaoFactory를 사용하는 애플리케이션 컨텍스트



```java
// 애플리케이션 컨텍스트 또는 빈 팩토리가 사용할 설정정보라는 표시
@Configuration
public class DaoFactory {
  
  	// 오브젝트 생성을 담당하는 IoC용 메소드라는 표시
    @Bean
    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }

    @Bean
    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
  
}
```

```java
public class UserDaoTest {
   
  	public static void main(String[] args) throws ClassNotFoundException, SQLException {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);

        ...
          
    }
  
}
```



### 1.5.2 애플리케이션 컨텍스트의 동작방식

- 그림 1-9 참고
- DaoFactory를 오브젝트 팩토리로 직접 사용했을 때와 비교해서 애플리케이션 컨텍스트를 사용했을 때 얻을 수 있는 장점
  - 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다.
  - 애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해준다.
  - 애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다.



### 1.5.3 스프링 IoC의 용어 정리

- 빈(bean)
  - 빈 또는 빈 오브젝트는 스프링이 IoC 방식으로 관리하는 오브젝트라는 뜻
  - 스프링을 사용하는 애플리케이션에서 만들어지는 모든 오브젝트가 다 빈은 아니다.
  - 스프링이 직접 그 생성과 제어를 담당하는 오브젝트만을 빈이라고 부른다.
- 빈 팩토리(bean factory)
  - 스프링의 IoC를 담당하는 핵심 컨테이너를 가리킨다.
  - 빈을 등록하고, 생성하고, 조회하고 돌려주고, 그 외에 부가적인 빈을 관리하는 기능을 담당한다.
- 애플리케이션 컨텍스트(application context)
  - 빈 팩토리를 확장한 IoC 컨테이너다.
  - 빈을 등록하고 관리하는 기본적인 빈 팩토리 기능에 더해서 스프링이 제공하는 각종 부가 서비스를 추가로 제공한다.
- 설정정보/설정 메타정보(configuration metadata)
  - 스프링의 설정정보는 컨테이너에 어떤 기능을 세팅하거나 조정하는 경우에도 사용하지만, 그보다 IoC 컨테이너에 의해 관리되는 애플리케이션 오브젝트를 생성하고 구성할 때 사용된다.
- 컨테이너(container) 또는 IoC 컨테이너
  - IoC 방식으로 빈을 관리한다는 의미에서 애플리케이션 컨텍스트나 빈 팩토리를 컨테이너 또는 IoC 컨테이너라고 한다.
- 스프링 프레임워크
  - 스프링 프레임워크는 IoC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 제공하는 모든 기능을 통틀어 말할 때 주로 사용한다.



## 1.6 싱글톤 레지스트리와 오브젝트 스코프

- 스프링의 애플리케이션 컨텍스트는 기존에 직접 만들었던 오브젝트 팩토리와는 중요한 차이점이 있다.
  - 스프링은 여러 번에 걸쳐 빈을 요청하더라도 매번 동일한 오브젝트를 돌려준다.
    - 매번 new에 의해서 새로운 UserDao가 만들어지지 않는다는 뜻이다.
- **동일성**(identity)과 **동등성**(equality)
  - 동일성은 == 연산자로, 동등성은 equals() 메소드를 이용해 비교
  - 동일성은 메모리상의 주소가 같은지 여부
  - 동등성은 오브젝트의 동등성 기준에 따라 두 오브젝트의 정보가 동등한지 판단
  - 두 개의 오브젝트가 동일하지는 않지만 동등한 경우가 있을 수 있다.



### 1.6.1 싱글톤 레지스트리로서의 애플리케이션 컨텍스트

- 애플리케이션 컨텍스트는 싱글톤을 저장하고 관리하는 싱글톤 레지스트리(singleton registry)
  - 스프링은 기본적으로 별다른 설정을 하지 않으면 내부에서 생성하는 빈 오브젝트를 모두 싱글톤으로 만든다.
  - 여기서 싱글톤이라는 것은 디자인 패턴에서 나오는 싱글톤 패턴과 비슷한 개념이지만 그 구현 방법은 확연히 다르다.



#### 서버 애플리케이션과 싱글톤

- 매번 클라이언트에서 요청이 올 때마다 데이터 엑세스 로직, 서비스 로직, 비즈니스 로직, 프레젠테이션 로직 등을 담당하는 오브젝트를 새로 만들어서 사용한다면 어떻게 될까?
  - 서버당 초당 수십에서 수백 번씩 브라우저나 여타 시스템으로부터 요청을 받아야 할텐데
- 서블릿은 자바 엔터프라이즈 기술의 가장 기본이 되는 서비스 오브젝트
  - 스펙에서 강제하진 않지만, 서블릿은 대부분 멀티스레드 환경에서 싱글톤으로 동작한다.
  - 서블릿 클래스당 하나의 오브젝트만 만들어두고, 사용자의 요청을 담당하는 여러 스레드에서 하나의 오브젝트를 공유해 동시에 사용한다.



#### 싱글톤 패턴의 한계

- private 생성자를 갖고 있기 때문에 상속할 수 없다.
  - 애플리케이션의 로직을 담고 있는 일반 오브젝트의 경우 싱글톤으로 만들었을 때 객체지향적인 설계의 정점(상속과 이를 이용한 다형성)을 적용하기 어렵다는 점은 심각한 문제다.
- 싱글톤은 테스트하기 힘들다.
- 서버환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다.
- 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다.

```java
public class UserDao {
  	
  	private static UserDao INSTANCE;
  
  	...
      
    private UserDao(ConnnectionMaker connnectionMaker) {
      	this.connectionMaker = connectionMaker;
    }
  
  	public static synchronized UserDao getInstance() {
      	// 기본 생성자를 private으로 묶으니 이제 ConnectionMaker 오브젝트를 넣어주는게 불가능해졌다.
      	if (INSTANCE == null) INSTANCE = new UserDao(???);
      	return INSTANCE;
    }
  
  	...
  	
}
```



#### 싱글톤 레지스트리

- 스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공하는데 그것이 바로 싱글톤 레지스트리다.
  - 싱글톤 레지스트리의 장점은 스태틱 메소드와 private 생성자를 사용해야 하는 비정상적인 클래스가 아니라 평범한 자바 클래스를 싱글톤으로 활용하게 해준다는 점이다.
  - 평범한 자바 클래스라도 IoC 방식의 컨테이너를 사용해서 생성과 관계설정, 사용 등에 대한 제어권을 컨테이너에게 넘기면 손쉽게 싱글톤 방식으로 만들어져 관리되게 할 수 있다.
  - 가장 중요한 것은 싱글톤 패턴과 달리 스프링이 지지하는 객체지향적인 설계 방식과 원칙, 디자인 패턴(싱글톤 패턴은 제외) 등을 적용하는 데 아무런 제약이 없다는 점이다.



### 1.6.2 싱글톤과 오브젝트의 상태

- 기본적으로 싱글톤이 멀티스레드 환경에서 서비스 형태의 오브젝트로 사용되는 경우에는 상태정보를 내부에 갖고 있지 않은 무상태(stateless) 방식으로 만들어져야 한다.
  - 상태가 없는 방식으로 클래스를 만드는 경우에 각 요청에 대한 정보나, DB나 서버의 리소스로부터 생성한 정보는 어떻게 다뤄야 할까?
    - 이때는 파라미터와 로컬 변수, 리턴 값 등을 이용하면 된다.
  - 동일하게 **읽기전용의 속성**을 가진 정보라면 싱글톤에서 **인스턴스 변수**로 사용해도 좋다.
    - 물론 단순히 읽기전용 값이라면 static final이나 final로 선언하는 편이 나을 것이다.



### 1.6.3 스프링 빈의 스코프

- 빈의 스코프
  - 스프링이 관리하는 오브젝트, 즉 빈이 생성되고, 존재하고, 적용되는 범위
  - 싱글톤 스코프
    - 스프링 빈의 기본 스코프는 싱클톤 스코프
  - 프로토타입 스코프
    - 컨테이너에 빈을 요청할 때마다 매번 새로운 오브젝트를 만들어준다.
  - 리퀘스트 스코프
    - 웹을 통해 새로운 HTTP 요청이 생길 때마다 생성되는 요청 스코프
  - 세션 스코프
    - 웹의 세션과 스코프가 유사한 세션 스코프



## 1.7 의존관계 주입(DI)



### 1.7.1 제어의 역전(IoC)과 의존관계 주입

- IoC라는 용어는 매우 느슨하게 정의돼서 폭넓게 사용되고 있다.
  - 스프링을 IoC 컨테이너라고만 해서는 스프링이 제공하는 기능의 특징을 명확하게 설명하지 못한다.
  - 스프링이 제공하는 IoC 방식의 핵심을 짚어주는 **의존 관계 주입**(**D**ependency **I**njection)이라는, 좀 더 의도가 명확히 드러나는 이름을 사용하기 시작함.
    - 스프링이 여타 프레임워크와 차별화돼서 제공해주는 기능은 의존관계 주입이라는 새로운 용어를 사용할 때 분명하게 드러난다.
    - 초기에는 IoC 컨테이너라고 불렸지만 지금은 의존관계 주입 컨테이너 또는 DI 컨테이너라고 더 많이 불린다. 



### 1.7.2 런타임 의존관계 설정



#### 의존관계

- A가 B에 의존하고 있다.
  - B가 변하면 그것이 A에 영향을 미친다는 뜻.
  - 의존관계에는 방향성이 있다.
    - A가 B에 의존하고 있지만, 반대로 B는 A에 의존하지 않는다.
      - B는 A의 변화에 영향을 받지 않는다.



#### UserDao의 의존관계





#### UserDao의 의존관계 주입



### 1.7.3 의존관계 검색과 주입



### 1.7.4 의존관계 주입의 응용



#### 기능 구현의 교환



#### 부가기능 추가



### 1.7.5 메소드를 이용한 의존관계 주입









## 1.8 XML을 이용한 설정

### 1.8.1 XML 설정

#### connectionMaker() 전환

#### userDao() 전환

#### XML의 의존관계 주입 정보

### 1.8.2 XML을 이용하는 애플리케이션 컨텍스트

### 1.8.3 DataSource 인터페이스로 변환

#### DataSource 인터페이스 적용

137

#### 자바 코드 설정 방식

#### XML 설정 방식

### 1.8.4 프로퍼티 값의 주입

#### 값 주입

#### value 값의 자동 변환









## 1.9 정리

142







