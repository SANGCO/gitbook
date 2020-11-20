> 스프링에 적용된 가장 인기 있는 AOP의 적용 대상은 바로 선언적 트랜잭션 기능이다. 서비스 추상화를 통해 많은 근본적인 문제를 해결했던 트랜잭션 경계설정 기능을 AOP를 이용해 더욱 세련되고 깔끔한 방식으로 바꿔보자. 그리고 그 과정에서 스프링이 AOP를 도입해야 했던 이유도 알아보자.



## 6.1 트랜잭션 코드의 분리



### 6.1.1 메소드 분리

- `리스트 6-1` 트랜잭션 경계설정과 비즈니스 로직이 공존하는 메소드
  - 트랜잭션 경계설정 코드와 비즈니스 로직 코드가 뚜렷하게 구분되어 있다. 
  - 비즈니스 로직 코드를 사이에 두고 트랜잭션 시작과 종료를 담당하는 코드가 앞뒤에 위치
  - 두 가지 코드는 성격이 다를 뿐 아니라 서로 주고받는 것도 없는, 완벽하게 독립적인 코드
- `리스트 6-2` 비즈니스 로직과 트랜잭션 경계설정의 분리



### 6.1.2 DI를 이용한 클래스의 분리



#### DI 적용을 이용한 트랜잭션 분리

- `그림 6-3` 트랜잭션 경계설정을 위한 UserServiceTx의 도입
  - UserServiceTx 클래스는 사용자 관리 로직을 담고 있지는 않고 단지 트랜잭션의 경계설정이라는 책임을 맡고 있을 뿐이다.
    - 사용자 관리 로직은 UserServiceImpl



#### UserService 인터페이스 도입



#### 분리된 트랜잭션 기능

```java
public class UserServiceTx implements UserService {
    UserService userService;
    PlatformTransactionManager transactionManager;

    public void setTransactionManager(
        PlatformTransactionManager transactionManager) {
     		this.transactionManager = transactionManager;
    }

    public void setUserService(UserService userService) {
      	this.userService = userService;
    }

    public void add(User user) {
      	this.userService.add(user);
    }

    public void upgradeLevels() {
        TransactionStatus status = this.transactionManager
            .getTransaction(new DefaultTransactionDefinition());
        try {

          	// 같은 인터페이스를 구현한 다른 오브젝트에게 고스란히 작업을 위임
            // UserServiceTx가 UserService 감쌌다.
            userService.upgradeLevels();

            this.transactionManager.commit(status);
        } catch (RuntimeException e) {
            this.transactionManager.rollback(status);
            throw e;
        }
    }
}
```



#### 트랜잭션 적용을 위한 DI 설정

- `그림 6-4` 트랜잭션 기능의 오브젝트가 적용된 의존관계
- `리스트 6-7` 트랜잭션 오브젝트가 추가된 설정파일



#### 트랜잭션 분리에 따른 테스트 수정

```java
@Test
public void upgradeAllOrNothing() {
    TestUserService testUserService = 
        new TestUserService(users.get(3).getId());
    testUserService.setUserDao(userDao);
    testUserService.setMailSender(mailSender);

    UserServiceTx txUserService = new UserServiceTx();
    txUserService.setTransactionManager(transactionManager);
    txUserService.setUserService(testUserService);

    userDao.deleteAll();			  
    for(User user : users) userDao.add(user);

    try {
        txUserService.upgradeLevels();   
        fail("TestUserServiceException expected"); 
    }
    
  	...
}
```



#### 트랜잭션 경계설정 코드 분리의 장점

- 비즈니스 로직을 담당하고 있는 UserServiceImpl의 코드를 작성할 때는 트랜잭션과 같은 기술적인 내용에는 전혀 신경 쓰지 않아도 된다.
- 비즈니스 로직에 대한 테스트를 손쉽게 만들어낼 수 있다.



## 6.2 고립된 단위 테스트

- 테스트는 작은 단위로 하면 좋다.
  - 테스트 대상이 다른 오브젝트와 환경에 의존하고 있다면 작은 단위의 테스트가 주는 장점을 얻기 힘들다.



### 6.2.1 복잡한 의존관계 속의 테스트



### 6.2.2 테스트 대상 오브젝트 고립시키기



#### 테스트를 위한 UserServiceImpl 고립



#### 고립된 단위 테스트 활용



#### UserDao 목 오브젝트



#### 테스트 수행 성능의 향상



### 6.2.3 단위 테스트와 통합 테스트



### 6.2.4 목 프레임워크



#### Mockito 프레임워크





## 6.3 다이내믹 프록시와 팩토리 빈



### 6.3.1 프록시와 프록시 패턴, 데코레이터 패턴

- 마치 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것을 대리자, 대리인과 같은 역할을 한다고 해서 프록시(proxy)라고 부른다.
- 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 타깃(target) 또는 실체(real subject)라고 부른다.



#### 데코레이터 패턴

- 데코레이터 패턴에서는 프록시가 꼭 한개로 제한되지 않는다.
  - 프록시가 여러 개인 만큼 순서를 정해서 단계적으로 위임하는 구조
- `그림 6-11` 데코레이터 패턴 적용 예
- 자바 IO 패키지의 InputStream과 OutputStream 구현 클래스는 데코레이터 패턴이 사용된 대표적인 예다.



#### 프록시 패턴

- 일반적으로 사용하는 프록시라는 용어와 디자인 패턴에서 말하는 프록시 패턴은 구분할 필요가 있다.
  - 전자는 클라이언트와 사용 대상 사이에 대리 역할을 맡은 오브젝트를 두는 방법을 총칭한다.
  - 후자는 프록시를 사용하는 방법 중에서 타깃에 대한 접근 방법을 제어하려는 목적을 가진 경우를 가리킨다.
    - 프록시 패턴은 타깃의 기능 자체에는 관여하지 않으면서 접근하는 방법을 제어해주는 프록시를 이용
- `그림 6-12` 프록시 패턴과 데코레이터 패턴의 혼용
  - 접근 제어를 위한 프록시를 두는 프록시 패턴과 컬러, 페이징 기능을 추가하기 위한 프록시를 두는 데코레이터 패턴을 함께 적용한 예
  - 두 가지 모두 프록시의 기본 윈리대로 타깃과 같은 인터페이스를 구현해두고 위임하는 방식으로 만들어져 있다.
- 앞으로는 타깃과 동일한 인터페이스를 구현하고 클라이언트와 타깃 사이에 존재하면서 **기능의 부가** 또는 **접근 제어**를 담당하는 오브젝트를 모두 **프록시**라고 부르겠다.
  - 사용의 목적이 기능의 부가인지, 접근 제어인지 구분해보면 어떤 목적으로 프록시가 사용됐는지, 그에 따라 어떤 패턴이 적용됐는지 알 수 있을 것이다.



### 6.3.2 다이내믹 프록시



#### 프록시의 구성과 프록시 작성의 문제점

- 프록시는 다음의 두 가지 기능으로 구성된다.
  - 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.
  - 지정된 요청에 대해서는 부가기능을 수행한다.
  - `리스트 6-16` UserServiceTx 프록시의 기능 구분
- 프록시를 만들기가 번거로운 이유
  - 첫 번째 타깃의 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거롭다.
  - 두 번째 부가기능 코드가 중복될 가능성이 많다.
    - 메소드가 많아지고 트랜잭션 적용의 비율이 높아지면 트랜잭션 기능을 제공하는 유사한 코드가 여러 메소드에 중복돼서 나타날 것이다.
- 두 번째 문제인 부가기능의 중복 문제는 중복되는 코드를 분리해서 어떻게든 해결해보면 될 것 같다.
  - 첫 번째 문제는?
    - JDK의 다이내믹 프록시로 해결



#### 리플렉션



#### 프록시 클래스

```java
// 리스트 6-18 Hello 인터페이스
interface Hello {
  	String sayHello(String name);
  	String sayHi(String name);
  	String sayThankYou(String name);
}
```

```java
// 리스트 6-19 타깃 클래스
public class HelloTarget implements Hello {
  	public String sayHello(String name) {
      	return "Hello " + naem;
    }
  
  	public String sayHi(String name) {
      	return "Hi " + naem;
    }
  
  	public String sayThankYou(String name) {
      	return "Thank You " + naem;
    }
}
```

```java
// 리스트 6-21 프록시 클래스
public class HelloUppercase implements Hello {
  	// 다른 프록시를 추가할 수도 있으니 인터페이스로 접근
  	Hello hello;
  
  	public HelloUppercase(Hello hello) {
      	this.hello = hello;
    }
  	
  	public String sayHello(String name) {
      	// 위임과 부가기능 적용
      	return hello.sayHello(name).toUpperCase();
    }
  
  	public String sayHi(String name) {
      	return hello.sayHi(name).toUpperCase();
    }
  
  	public String sayThankYou(String name) {
      	return hello.sayThankYou(name).toUpperCase();
    }
}
```

- 이 프록시는 프록시 적용의 일반적인 문제점 두 가지를 모두 갖고 있다.
  - 인터페이스의 모든 메소드를 구현해 위임하도록 코드를 만들어야 한다.
  - 부가기능인 리턴 값을 대문자로 바꾸는 기능이 모든 메소드에 중복돼서 나타난다.



#### 다이내믹 프록시 적용

- `그림 6-13` 다이내믹 프록시의 동작방식
  - `클라이언트`의 요청에 `프록시 팩토리`가 `다이내믹 프록시`를 생성
  - `클라이언트`가 `다이내믹 프록시`를 호출
  - `다이내믹 프록시`가 `InvocationHandler` 호출
  - `InvocationHandler`는 `타깃`에 위임
- 위에 Hello 예제와 비교
  - HelloUppercase 클래스를 만들 필요가 없다.
    - HelloTarget 클래스는 만들어야한다.
  - 메소드 별로 부가기능을 적용할 필요없이 InvocationHandler 한군데만 작업하면 된다.
- 다이내믹 프록시는 프록시 팩토리에 의해 런타임 시 다이내믹하게  만들어지는 오브젝트다.
  - 프록시 팩토리에게 인터페이스 정보만 제공해주면 해당 인터페이스를 구현한 클래스의 오브젝트를 자동으로 만들어주기 때문이다.
- 부가기능은 프록시 오브젝트와 독립적으로 InvocationHandler를 구현한 오브젝트에 담는다.
  - 타깃 인터페이스의 모든 메소드 요청이 하나의 메소드로 집중되기 때문에 중복되는 기능을 효과적으로 제공할 수 있다.
- `그림 6-14` InvocationHandler를 통한 요청 처리 구조
  - Hello 인터페이스를 제공하면서 프록시 팩토리에게 다이내믹 프록시를 만들어달라고 요청
    - Hello 인터페이스의 모든 메소드를 구현한 오브젝트를 생성해준다.
  - InvocationHandler 인터페이스를 구현한 오브젝트를 제공
    - 다이내믹 프록시가 받는 모든 요청을 InvocationHandler의 invoke() 메소드로 보내준다.
    - Hello 인터페이스의 메소드가 아무리 많더라도 invoke() 메소드 하나로 처리할 수 있다.

```java
// 리스트 6-23 InvocationHandler 구현 클래스
public class UppercaseHandler implements InvocationHandler {
  	Hello target;
  
  	public UppercaseHandler(Hello target) {
      	this.target = target;
    }
  
  	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      	// 타깃으로 위임. 인터페이스의 메소드 호출에 모두 적용된다.
      	String ret = (String)method.invoke(target, args);
      	return ret.toUpperCase(); // 부가기능 제공
    }
}
```

```java
// 리스트 6-24 프록시 생성
Hello proxiedHello = (Hello) Proxy.newProxyInstance(
		    getClass().getClassLoader(),
  			new Class[] { Hello.class },
  			new UppercaseHandler(new HelloTarget())
		);
```



#### 다이내믹 프록시의 확장

- `리스트 6-25` 확장된 UppercaseHnadler
  - 부가기능을 handler로
  - 오브젝트의 메소드 호출 후 리턴 타입을 확인해서 리턴 타입이 스트링인 경우만 대문자로 바꿔줄 수 있다. 
  - 어떤 종류의 인터페이스를 구현한 타깃이든 상관없이 재사용할 수 있다.
    - target의 타입을 Object로



- JAVA Proxy 예제 만든거 참고

```java
public interface Multiply {

    int twice(int x);

    int treble(int x);

}
```

```java
public class MultiplyImpl implements Multiply {

    @Override
    public int twice(int x) {
        return x * 2;
    }

    @Override
    public int treble(int x) {
        return x * 3;
    }
}
```

```java
public class JavaProxyHandler implements InvocationHandler {

    private Object target;

    public JavaProxyHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("메소드 시작");
        int result = (int) method.invoke(target, args);
        System.out.println("메소드 끝");
        return result;
    }
}
```

```java
public class Main {

    public static void main(String[] args) {
        Multiply twice = new MultiplyImpl();
        Class<? extends Multiply> twiceClass = twice.getClass();
        Multiply proxyInstance = (Multiply) Proxy.newProxyInstance(
            twiceClass.getClassLoader(),
            twiceClass.getInterfaces(),
            new JavaProxyHandler(twice)
        );
      	// twice()와 treble()이 호출 될 때마다 JavaProxyHandler의 invoke()가 호출된다.
      	// invoke() 메소드의 파라미터 Method에 호출하는 메소드의 정보가 들어있다.
        System.out.println(proxyInstance.twice(5));
        System.out.println(proxyInstance.treble(5));
    }

}
```



### 6.3.3 다이내믹 프록시를 이용한 트랜잭션 부가기능



#### 트랜잭션 InvocationHandler



#### TransactionHandler와 다이내믹 프록시를 이용하는 테스트



### 6.3.4 다이내믹 프록시를 위한 팩토리 빈

- 앞 절에서는 어떤 타깃에도 적용 가능한 트랜잭션 부가기능을 담은 TransactionHandler를 만들고, 이를 이용하는 다이내믹 프록시를 UserService에 적용하는 테스트를 만들어봤다.
- 이제 TransactionHandler와 다이내믹 프록시를 스프링의 DI를 통해 사용할 수 있도록 만들어야 할 차례다.
  - 그런데 문제는 DI의 대상이 되는 다이내믹 프록시 오브젝트는 일반적인 스프링의 빈으로는 등록할 방법이 없다는 점이다.
- 스프링의 빈은 기본적으로 클래스 이름과 프로퍼티로 정의된다.
- 스프링은 지정된 클래스 이름을 가지고 리플렉션을 이용해서 해당 클래스의 오브젝트를 생성할 수 있다.
  - 스프링은 내부적으로 리플렉션 API를 이용해서 빈 정의에 나오는 클래스 이름을 가지고 빈 오브젝트를 생성한다.
    - Class의 newInstance() 메소드는 해당 클래스의 파라미터가 없는 생성자를 호출하고, 그 결과 생성되는 오브젝트를 돌려주는 리플렉션 API다.
    - `Date now = (Date) Class.forName("java.util.Date").newInstance();`



#### 팩토리 빈

- 사실 스프링은 클래스 정보를 가지고 디폴트 생성자를 통해 오브젝트를 만드는 방법 외에도 빈을 만들 수 있는 여러 가지 방법을 제공한다.
  - 대표적으로 팩토리 빈을 이용한 빈 생성 방법을 들 수 있다.
    - 팩토리 빈이란 스프링을 대신해서 오브젝트의 생성로직을 담당 하도록 만들어진 특별한 빈을 말한다.
    - 팩토리 빈을 만드는 방법에는 여러 가지가 있는데, 가장 간단한 방법은 스프링의 FactoryBean이라는 인터페이스를 구현하는 것이다.



#### 팩토리 빈의 설정 방법

 ```java
public class TxProxyFactoryBean implements FactoryBean<Object> {
    Object target;
    PlatformTransactionManager transactionManager;
    String pattern;
    Class<?> serviceInterface;

    ...

    public Object getObject() throws Exception {
        TransactionHandler txHandler = new TransactionHandler();
      	
        ...
      
        return Proxy.newProxyInstance(
            getClass().getClassLoader(),
            new Class[] { serviceInterface }, 
            txHandler
        );
    }

    ...

}      
 ```



#### 다이내믹 프록시를 만들어주는 팩토리 빈

- `그림 6-15` 팩토리 빈을 이용한 트랜잭션 다이내믹 프록시의 적용
  - 팩토리 빈을 사용하면 다이내믹 프록시 오브젝트를 스프링의 빈으로 만들어줄 수가 있다.



#### 트랜잭션 프록시 팩토리 빈



#### 트랜잭션 프록시 팩토리 빈 테스트

- TxProxyFactoryBean 생성해서 getObject() 메소드 호출

  



### 6.3.5 프록시 팩토리 빈 방식의 장점과 한계



#### 프록시 팩토리 빈의 재사용

- `그림 6-16` 설정 변경을 통한 트랜잭션 기능 부가



#### 프록시 팩토리 빈 방식의 장점

- 다이내믹 프록시에 팩토리 빈을 이용한 DI까지 더해주면 번거로운 다이내믹 프록시 생성 코드도 제거할 수 있다.
  - DI 설정만으로 다양한 타깃 오브젝트에 적용도 가능하다.
  - 프록시 팩토리 빈을 좀 더 효과적으로 사용하고자 할 때도 DI가 중요한 역할을 한다.



#### 프록시 팩토리 빈의 한계

- 한 번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 일은 지금까지 살펴본 방법으로는 불가능하다.

  - 한 번에 하나의 타겟

  - `txProxyFactoryBean.setTarget(testUserService);`

- 하나의 타깃에 여러 개의 부가기능을 적용하려고 할 때도 문제다.

  - TransactionHandler 처럼 한 번에 하나의 부가기능

- TransactionHandler 오브젝트가 프록시 팩토리 빈 개수만큼 만들어진다.

  - ```java
    TransactionHandler txHandler = new TransactionHandler();   	
    ...
    return Proxy.newProxyInstance(
        getClass().getClassLoader(),
        new Class[] { serviceInterface }, 
        txHandler
    ```



## 6.4 스프링의 프록시 팩토리 빈

- 이제 스프링은 이러한 문제에 어떤 해결책을 제시하는지 살펴볼 차례다.
  - 언제나 그렇지만 스프링은 매우 세련되고 깔끔한 방식으로 애플리케이션 개발에 자주 등장하는 이런 문제에 대한 해법을 제공한다.



### 6.4.1 ProxyFactoryBean

- 자바에는 JDK에서 제공하는 다이내믹 프록시 외에도 편리하게 프록시를 만들 수 있도록 지원해주는 다양한 기술이 존재한다.
  - 따라서 스프링은 일관된 방법으로 프록시를 만들 수 있게 도와주는 추상 레이어를 제공한다.
    - 생성된 프록시는 스프링의 빈으로 등록돼야 한다.
    - 스프링은 프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈을 제공해준다.
- 스프링의 ProxyFactoryBean은 프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈이다.
- ProxyFactoryBean이 생성하는 프록시에서 사용할 부가기능은 MethodInterceptor 인터페이스를 구현해서 만든다.
  - MethodInterceptor의 invoke() 메소드는 ProxyFactoryBean으로부터 타깃 오브젝트에 대한 정보까지도 함께 제공받는다.
    - **이 차이 덕분에 MethodInterceptor는 타깃 오브젝트에 상관없이 독립적으로 만들어질 수 있다.**
    - MethodInterceptor 오브젝트는 타깃이 다른 여러 프록시에서 함께 사용할 수 있고, 싱글톤 빈으로 등록 가능하다.

```java
@Test
public void proxyFactoryBean() {
    ProxyFactoryBean pfBean = new ProxyFactoryBean();
    pfBean.setTarget(new HelloTarget());
    pfBean.addAdvice(new UppercaseAdvice());

    Hello proxiedHello = (Hello) pfBean.getObject();

    assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
    assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
    assertThat(proxiedHello.sayThankYou("Toby"), is("THANK YOU TOBY"));
}

static class UppercaseAdvice implements MethodInterceptor {
    public Object invoke(MethodInvocation invocation) throws Throwable {
        // 리플렉션의 Method와 달리 메소드 실행 시 타깃 오브젝트를 전달할 필요가 없다.
        // MethodInvocation은 메소드 정보와 함께 타깃 오브젝트를 알고 있기 때문이다.
      	String ret = (String)invocation.proceed();
        return ret.toUpperCase();
    }
}
```



#### 어드바이스: 타깃이 필요 없는 순수한 부가기능

- MethodInvocation은 일종의 콜백 오브젝트로, proceed() 메소드를 실행하면 타깃 오브젝트의 메소드를 내부적으로 실행해주는 기능이 있다.
- ProxyFactoryBean은 작은 단위의 템플릿/콜백 구조를 응용해서 적용했기 때문에 템플릿 역할을 하는 MethodInvocation을 싱글톤으로 두고 공유할 수 있다.
  - 마치 SQL 파라미터 정보에 종속되지 않는 JdbcTemplate이기 때문에 수많은 DAO 메소드가 하나의 JdbcTemplate 오브젝트를 공유할 수 있는 것과 마찬가지다.
- **ProxyFactoryBean 하나만으로 여러 개의 부가기능을 제공해주는 프록시를 만들 수 있다.**
  - addAdvice()
  - add라는 이름에서 알 수 있듯이 ProxyFactoryBean에는 여러 개의 MethodInterceptor를 추가할 수 있다.
- MethodInterceptor처럼 타깃 오브젝트에 적용하는 부가기능을 담은 오브젝트를 스프링에서는 어드바이스(advice)라고 부른다.



#### 포인트컷: 부가기능 적용 대상 메소드 선정 방법

- 스프링은 부가기능을 제공하는 오브젝트를 어드바이스라고 부르고, 메소드 선정 알고리즘을 담은 오브젝트를 포인트컷이라고 부른다.
- `그림 6-17` 기존 JDK 다이내믹 프록시를 이용한 방식
- `그림 6-18` 스프링 ProxyFactoryBean을 이용한 방식
  - ProxyFactoryBean 방식은 두 가지 확장 기능인 부가기능(Advice)과 선정 알고리즘(Pointcut)을 활용하는 유연한 구조를 제공한다.
    - 부기기능 적용 메소드를 선택하는 기능을 프록시에 두지 않고 분리 시켰다.
  - 프록시는 클라이언트로부터 요청을 받으면 먼저 포인트컷에게 부가기능을 부여할 메소드인지를 확인해달라고 요청한다.
- 어드바이스가 부가기능을 부여하는 중에 타깃 메소드의 호출이 필요하면 프록시부터 전달받은 MethodInvocation 타입 콜백 오브젝트의 proceed() 메소드를 호출해주기만 하면 된다.
- **재사용 가능한 기능을 만들어두고 바뀌는 부분(콜백 오브젝트와 메소드 호출정보)만 외부에서 주입해서 이를 작업 흐름(부가기능 부여) 중에 사용하도록 하는 정형적인 템플릿/콜백 구조다.**
  - 어드바이스가 일종의 템플릿이 되고 타깃을 호출하는 기능을 갖고 있는 MethodInvocation 오브젝트가 콜백이 되는 것이다.
- 프록시로부터 어드바이스와 포인트컷을 독립시키고 DI를 사용하게 한 것은 전형적인 전략 패턴 구조다.
- 포인트컷과 어드바이스를 따로 등록하면 어떤 어드바이스(부가기능)에 대해 어떤 포인트컷(메소드 선정)을 적용할지 애매진다.
  - 어드바이스와 포인트컷을 묶은 오브젝트를 인터페이스 이름을 따서 어드바이저라고 부른다.
  - `어드바이저 = 포인터컷(메소드 선정 알고리즘) + 어드바이스(부가기능)`



### 6.4.2 ProxyFactoryBean 적용



#### TransactionAdvice



#### 스프링 XML 설정파일

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />  
</bean>

<bean id="transactionAdvice" class="springbook.user.service.TransactionAdvice">
		<property name="transactionManager" ref="transactionManager" />
</bean>

<bean id="transactionPointcut" class="org.springframework.aop.support.NameMatchMethodPointcut">
		<property name="mappedName" value="upgrade*" />
</bean>

<bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
		<property name="advice" ref="transactionAdvice" />
		<property name="pointcut" ref="transactionPointcut" />
</bean>

<!--  application components -->

<bean id="userService" class="org.springframework.aop.framework.ProxyFactoryBean">
		<property name="target" ref="userServiceImpl" />
		<property name="interceptorNames">
				<list>
						<value>transactionAdvisor</value>
				</list>
		</property>
</bean>
```




#### 테스트



#### 어드바이스와 포인트컷의 재사용

- ProxyFactoryBean은 스프링의 DI와 템플릿/콜백 패턴, 서비스 추상화 등의 기법이 모두 적용된 것이다.
  - 그 덕분에 독립적이며, 여러 프록시가 공유할 수 있는 어드바이스와 포인트컷으로 확장 기능을 분리할 수 있었다.
- `그림 6-19` ProxyFactoryBean, Advice, Pointcut을 적용한 구조
  - 그림 보면 더 헤깔리네. 위에 XML 파일을 보자.



## 6.5 스프링 AOP



### 6.5.1 자동 프록시 생성

- 새로운 타깃이 등장했다고 해서 코드를 손댈 필요는 없어졌지만, 설정은 매번 복사해서 붙이고 target 프로퍼티의 내용을 수정해줘야 한다.



#### 중복 문제의 접근 방법

- 의미 있는 부가기능 로직인 트랜잭션 경계설정은 코드로 만들게 하고, 기계적인 코드인 타깃 인터페이스 구현과 위임, 부가기능 연동 부분은 자동생성

- 지금까지 살펴본 방법에서는 한 번에 여러 개의 빈에 프록시를 적용할 만한 방법은 없었다.

  - ```java
    ProxyFactoryBean pfBean = new ProxyFactoryBean();
    pfBean.setTarget(new HelloTarget());
    ```



#### 빈 후처리기를 이용한 자동 프록시 생성기

- 스프링은 컨테이너로서 제공하는 기능 중에서 변하지 않는 핵심적인 부분 외에는 대부분 확장할 수 있도록 확장 포인트를 제공해준다.
  - 그중에서 관심을 가질 만한 확장 포인트는 바로 BeanPostProcessor 인터페이스를 구현해서 만드는 빈 후처리기다.
    - 빈 후처리기는 이름 그대로 스프링 빈 오브젝트로 만들어지고 난 후에, 빈 오브젝트를 다시 가공할 수 있게 해준다.

- `DefaultAdvisorAutoProxyCreator`
  - 어드바이저를 이용한 자동 프록시 생성기
  - 빈 후처리기 자체를 빈으로 등록
  - 스프링은 빈 후처리기가 빈으로 등록되어 있으면 빈 오브젝트가 생성될 때마다 빈 후처리기에 보내서 후처리 작업을 요청한다.
    - 빈 오브젝트의 프로퍼티를 강제로 수정 가능
    - 별도의 초기화 작업 수행 가능
    - 만들어진 빈 오브젝트 자체를 바꿔치기도 가능
  - **이를 잘 이용하면 스프링이 생성하는 빈 오브젝트의 일부를 프록시로 포장하고, 프록시를 빈으로 대신 등록할 수도 있다.**
- `그림 6-20` 빈 후처리기를 이용한 프록시 자동생성
  - 스프링은 빈 오브젝트를 만들 때마다 후처리기에게 빈을 보낸다.
  - 모든 어드바이저 내의 포인트컷을 이용해 전달받은 빈이 프록시 적용 대상인지 확인
  - 프록시 적용 대상이면 프록시 만들고 어드바이저를 연결
  - 빈 후처리기는 컨테이너가 전달해준 빈 오브젝트 대신 프록시 오브젝트를 컨테이너에게 돌려준다.
- 적용할 빈을 선정하는 로직이 추가된 포인트컷이 담긴 어드바이저를 등록하고 빈 후처리기를 사용하면 일일이 ProxyFactoryBean 빈을 등록하지 않아도 타깃 오브젝트에 자동으로 프록시가 적용되게 할 수 있다.



#### 확장된 포인트컷

```java
// 리스트 6-49 두 가지 기능을 정의한 Pointcut 인터페이스
public interface Pointcut {
  	// 프록시를 적용할 클래스인지 확인해준다.
  	ClassFilter getClassFilter();
  	// 어드바이스를 적용할 메소드인지 확인해준다.
  	MethodMatcher getMethodMatcher();
}
```

- Pointcut 선정 기능을 모두 적용한다면 먼저 프록시를 적용할 클래스인지 판단하고 나서, 적용 대상 클래스인 경우에는 어드바이저를 적용할 메소드인지 확인하는 식으로 동작한다.
- 모든 빈에 대해 프록시 자동 적용 대상을 선별해야 하는 빈 후처리기인 DefaultAdvisorAutoProxyCreator는 클래스와 메소드 선정 알고리즘을 모두 갖고 있는 포인트컷이 필요하다.
  - 정확히는 그런 포인트컷과 어드바이스가 결합되어 있는 어드바이저가 등록되어 있어야 한다.
  - ProxyFactoryBean에서는 굳이 클래스 레벨의 필터는 필요 없었다.



#### 포인트컷 테스트





### 6.5.2 DefaultAdvisorAutoProxyCreator의 적용



#### 클래스 필터를 적용한 포인트컷 작성



#### 어드바이저를 이용하는 자동 프록시 생성기 등록



#### 포인트컷 등록



#### 어드바이스와 어드바이저



#### ProxyFactoryBean 제거와 서비스 빈의 원상복구



#### 자동 프록시 생성기를 사용하는 테스트



#### 자동생성 프록시 확인



### 6.5.3 포인트컷 표현식을 이용한 포인트컷



#### 포인트컷 표현식



#### 포인트컷 표현식 문법



#### 포인트컷 표현식 테스트



#### 포인트컷 표현식을 이용하는 포인트컷 적용



#### 타입 패턴과 클래스 이름 패턴



### 6.5.4 AOP란 무엇인가?



#### 트랜잭션 서비스 추상화



#### 프록시와 데코레이터 패턴



#### 다이내믹 프록시와 프록시 팩토리 빈



#### 자동 프록시 생성 방법과 포인트컷



#### 부가기능의 모듈화



#### AOP: 애스펙트 지향 프로그래밍





### 6.5.5 AOP 적용기술



#### 프록시를 이용한 AOP



#### 바이트코드 생성과 조작을 통한 AOP



### 6.5.6 AOP의 용어



### 6.5.7 AOP 네임스페이스



#### AOP 네임스페이스



#### 어드바이저 내장 포인트컷











## 6.6 트랜잭션 속성



### 6.6.1 트랜잭션 정의



#### 트랜잭션 전파



#### 격리수준



#### 제한시간



#### 읽기전용



### 6.6.2 트랜잭션 인터셉터와 트랜잭션 속성



#### TransactionInterceptor



#### 메소드 이름 패턴을 이용한 트랜잭션 속성 지정



#### tx 네임스페이스를 이용한 설정 방법





### 6.6.3 포인트컷과 트랜잭션 속성의 적용 전략



#### 트랜잭션 포인트컷 표현식은 타입 패턴이나 빈 이름을 이용한다



#### 공통된 메소드 이름 규칙을 통해 최소한의 트랜잭션 어드바이스와 속성을 정의한다



#### 프록시 방식 AOP는 같은 타깃 오브젝트 내의 메소드를 호출할 때는 적용되지 않는다







### 6.6.4 트랜잭션 속성 적용



#### 트랜잭션 경계설정의 일원화



#### 서비스 빈에 적용되는 포인트컷 표현식 등록



#### 트랜잭션 속성을 가진 트랜잭션 어드바이스 등록



#### 트랜잭션 속성 테스트





## 6.7 애노테이션 트랜잭션 속성과 포인트컷



### 6.7.1 트랜잭션 애노테이션



#### @Transactional



#### 트랜잭션 속성을 이용하는 포인트컷



#### 대체 정책



#### 트랜잭션 애노테이션 사용을 위한 설정





### 6.7.2 트랜잭션 애노테이션 적용





## 6.8 트랜잭션 지원 테스트



### 6.8.1 선언적 트랜잭션과 트랜잭션 전파 속성



### 6.8.2 트랜잭션 동기화와 테스트



#### 트랜잭션 매니저와 트랜잭션 동기화



#### 트랜잭션 매니저를 이용한 테스트용 트랜잭션 제어



#### 트랜잭션 동기화 검증



#### 롤백 테스트





### 6.8.3 테스트를 위한 트랜잭션 애노테이션



#### @Transactional



#### @Rollback



#### @TransactionConfiguration



#### NotTransactional과 Propagation.NEVER



#### 효과적인 DB 테스트





## 6.9 정리



