

## 1장 IoC 컨테이너와 DI




## 1.1 IoC 컨테이너: 빈 팩토리와 애플리케이션 컨텍스트

- IoC (Inversion of Control)
  - 오브젝트의 생성과 관계설정, 사용, 제거
    - 스프링 애플리케이션에서는 애플리케이션 코드 대신 독립된 컨테이너가 담당한다.
      - 컨테이너가 코드 대신 오브젝트에 대한 제어권을 갖는다.

- 스프링 컨테이너를 IoC 컨테이너라고도 한다.
- 스프링에서 IoC를 담당하는 컨테이너
  - 빈 팩토리
    - 오브젝트의 생성과 오브젝트 사이의 런타임 관계를 설정하는 DI 관점으로 볼 때는 컨테이너를 빈 팩토리라고 한다.
  - 애플리케이션 컨텍스트
    - DI를 위한 빈 팩토리에 엔터프라이즈 애플리케이션을 개발하는 데 필요한 여러 가지 컨테이너 기능을 추가한 것을 애플리케이션 컨텍스트라고 한다.
- 스프링의 IoC 컨테이너는 일반적으로 애플리케이션 컨텍스트를 말한다.
- `ApplicationContext` 인터페이스는 `BeanFactory` 인터페이스를 상속한 서브인터페이스다.
  - 실제로 스프링 컨테이너 또는 IoC 컨테이너라고 부르는 것은 `ApplicationContext` 인터페이스를 구현한 클래스의 오브젝트다.
- 스프링 애플리케이션은 최소한 하나 이상의 IoC 컨테이너, 즉 애플리케이션 컨텍스트 오브젝트를 갖고 있다.



### 1.1.1 IoC 컨테이너를 이용해 애플리케이션 만들기

- `StaticApplicationContext ac = new StaticApplicationContext();`
  - 가장 간단하게 IoC 컨테이너를 만들었다.
  - 컨테이너가 본격적인 IoC 컨테이너로서 동작하려면 두 가지가 필요하다.
    - POJO 클래스
    - 설정 메타정보



#### POJO 클래스

- 각자 기능에 충실하게 독립적으로 설계된 POJO 클래스를 만들고, 결합도가 낮은 유연한 관계를 가질 수 있도록 인터페이스를 이용해 연결
  - IoC 컨테이너가 사용할 POJO를 준비하는 첫 단계
  - Hello 클래스에 Printer 인터페이스 타입 객체를 DI



#### 설정 메타정보

- 스프링 컨테이너가 관리하는 오브젝트는 빈이라고 부른다.
- IoC 컨테이너가 필요로 하는 설정 메타정보란
  - 빈을 어떻게 만들고 어떻게 동작하게 할 것인가에 관한 정보
- 스프링의 설정 메타정보는 `BeanDefinition` 인터페이스로 표현되는 순수한 추상 정보다.
  - `BeanDefinitionReader` 인터페이스
    - 원본의 포맷과 구조, 자료의 특성에 맞게 읽어와 `BeanDefinition` 오브젝트로 변환해 준다.
  - 스프링의 메타정보는 특정한 파일 포맷이나 형식에 제한되거나 종속되지 않는다.
    - XML
    - 소스코드 애노테이션
    - 자바 코드
    - 프로퍼티 파일
- 애플리케이션 컨텍스트는 `BeanDefinition`으로 만들어진 메타정보를 담은 오브젝트를 사용해 IoC와 DI 작업을 수행한다.
  - `그림 1-2` IoC 컨테이너를 통해 애플리케이션이 만들어지는 방식 참고
- `BeanDefinition` 인터페이스로 정의되는, IoC 컨테이너가 사용하는 빈 메타정보
  - 빈 아이디, 이름, 별칭
    - 빈 오브젝트를 구분할 수 있는 식별자
  - 클래스 또는 클래스 이름
    - 빈으로 만들 POJO 클래스 또는 서비스 클래스 정보
  - 스코프
    - 싱글톤, 프로토타입과 같은 빈의 생성 방식과 존재 범위
  - 프로퍼티 값 또는 참조
    - DI에 사용할 프로퍼티 이름과 값 또는 참조하는 빈의 이름
  - 생성자 파라미터 값 또는 참조
    - DI에 사용할 생성자 파라미터 이름과 값 또는 참조할 빈의 이름
  - 지연된 로딩 여부, 우선 빈 여부, 자동와이어링 여부, 부모 빈 정보, 빈팩토리 이름 등



```java
// 리스트 1-5 Hello 클래스 빈 등록
StaticApplicationContext ac = new StaticApplicationContext();
ac.registerSingleton("hello1", Hello.class); //빈 이름과 POJO 클래스 제공

Hello hello1 = ac.getBean("hello1", Hello.class);
assertThat(hello1, is(notNullValue()));
```

- 스프링 빈 메타정보의 항목들은 대부분 디폴트 값이 있다.
  - 싱글톤으로 관리되는 빈 오브젝트를 등록할 때 반드시 제공해줘야 하는 정보는 빈 이름과 POJO 클래스뿐이다.
  - 엥간한 메타정보는 Hello.class 같은 클래스 타입인 클래스 리터럴에 있을거 같은디

- IoC 컨테이너가 관리하는 빈은 오브젝트 단위지 클래스 단위가 아니다.
  - 같은 클래스의 빈을 여러 개 등록하고 빈마다 다른 설정을 지정해두고 사용할 수도 있다.



```java
// 리스트 1-6 BeanDefinition을 이용한 빈 등록
BeanDefinition helloDef = new RootBeanDefinition(Hello.class);
helloDef.getPropertyValues().addPropertyValue("name", "Spring");
ac.registerBeanDefinition("hello2", helloDef);

Hello hello2 = ac.getBean("hello2", Hello.class);
assertThat(hello2.sayHello(), is("Hello Spring"));
assertThat(hello1, is(not(hello2))); // 동일한 타입이지만 별개의 오브젝트임.
assertThat(ac.getBeanFactory().getBeanDefinitionCount(), is(2)); // 빈 설정 메타정보를 가져올 수 있다.
```

- 리스트 1-6은 직접 `BeanDefinition` 타입의 설정 메타정보를 만들어서 IoC 컨테이너에 등록하고 있다. 



### 1.1.2 IoC 컨테이너의 종류와 사용 방법

StaticApplicationContext

GenericApplicationContext

GenericXmlApplicationContext

WebApplicationContext





### 1.1.3 IoC 컨테이너 계층구조

부모 컨텍스트를 이용한 계층구조 효과

컨텍스트 계층구조 테스트





### 1.1.4 웹 애플리케이션의 IoC 컨테이너 구성

웹 애플리케이션의 컨텍스트 계층구조

웹 애플리케이션의 컨텍스트 구성 방법

루트 애플리케이션 컨텍스트 등록

서블릿 애플리케이션 컨텍스트 등록





## 1.2 IoC/DI를 위한 빈 설정 메타정보 작성



### 1.2.1 빈 설정 메타정보

빈 설정 메타정보 항목

### 1.2.2 빈 등록 방법

XML: 《bean》 태그

XML: 네임스페이스와 전용 태그

자동인식을 이용한 빈 등록: 스테레오타입 애노테이션과 빈 스캐너

자바 코드에 의한 빈 등록: @Configuration 클래스의 @Bean 메소드

자바 코드에 의한 빈 등록: 일반 빈 클래스의 @Bean 메소드

빈 등록 메타정보 구성 전략

### 1.2.3 빈 의존관계 설정 방법

XML: 《property》, 《constructor-arg》

XML: 자동와이어링

XML: 네임스페이스와 전용 태그

애노테이션: @Resource

애노테이션: @Autowired/@Inject

@Autowired와 getBean(), 스프링 테스트

자바 코드에 의한 의존관계 설정

빈 의존관계 설정 전략

### 1.2.4 프로퍼티 값 설정 방법

메타정보 종류에 따른 값 설정 방법

PropertyEditor와 ConversionService

컬렉션

Null과 빈 문자열

프로퍼티 파일을 이용한 값 설정

### 1.2.5 컨테이너가 자동등록하는 빈

ApplicationContext, BeanFactory

ResourceLoader, ApplicationEventPublisher

systemProperties, systemEnvironment

## 1.3 프로토타입과 스코프


### 1.3.1 프로토타입 스코프

프로토타입 빈의 생명주기와 종속성

프로토타입 빈의 용도

DI와 DL

프로토타입 빈의 DL 전략

### 1.3.2 스코프

스코프의 종류

스코프 빈의 사용 방법

커스텀 스코프와 상태를 저장하는 빈 사용하기


## 1.4 기타 빈 설정 메타정보


### 1.4.1 빈 이름

XML 설정에서의 빈 식별자와 별칭

애노테이션에서의 빈 이름

### 1.4.2 빈 생명주기 메소드

초기화 메소드

제거 메소드

### 1.4.3 팩토리 빈과 팩토리 메소드


## 1.5 스프링 3.1의 Ioc 컨테이너와 DI

### 1.5.1 빈의 역할과 구분

빈의 종류

컨테이너 인프라 빈과 전용 태그

빈의 역할

### 1.5.2 컨테이너 인프라 빈을 위한 자바 코드 메타정보

IoC/DI 설정 방법의 발전

자바 코드를 이용한 컨테이너 인프라 빈 등록

### 1.5.3 웹 애플리케이션의 새로운 IoC 컨테이너 구성

### 1.5.4 런타임 환경 추상화와 프로파일

환경에 따른 빈 설정정보 변경 전략과 한계

런타임 환경과 프로파일

활성 프로파일 지정 방법

프로파일 활용 전략

### 1.5.5 프로퍼티 소스

프로퍼티

스프링에서 사용되는 프로퍼티의 종류

프로파일의 통합과 추상화

프로퍼티 소스의 사용

@PropertySource와 프로퍼티 파일

웹 환경에서 사용되는 프로퍼티 소스와 프로퍼티 소스 초기화 오브젝트

## 1.6 정리



