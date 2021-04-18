

## 9장 두 번째 양파 껍질을 벗기기 위한 중간 점검



### 9.1 자체 점검 요구사항(필수)



### 9.2 자체 점검 요구사항(선택)



### 9.3 자체 점검 확인



## 10장 새로운 MVC 프레임워크 구현을 통한 점진적 개선

### 10.1 MVC 프레임워크 요구사항 3단계



### 10.2 MVC 프레임워크 구현 3단계

- WebApplicationInitializer를 구현한 클래스를 만들어두면 웹 애플리케이션이 시작될 때 onStartup() 메소드가 자동으로 실행된다. 이때 메소드 파라미터로 ServletContext 오브젝트가 전달되는데, 이를 이용해 필요한 컨텍스트 등록 작업을 수행하면 된다.

- MyWebApplicationInitializer

  - AnnotationHandlerMapping 초기화

    - @Controller 어노테이션 붙인 클래스 가지고 온다.

    - 거기서 @RequestMapping 어노테이션 붙인 메소드 가지고 온다.

    - ```java
      // 여기에 HandlerKey, HandlerExecution 생성해서 넣어준다.
      Map<HandlerKey, HandlerExecution> handlerExecutions = Maps.newHashMap();
      ```

      - HandlerKey
        - URL, RequestMethod
        - 초기화 할 때는 @RequestMapping 어노테이션의 value(), method()에서 정보를 가지고 와서 만든다.
        - 요청이 들어오면 Request에서 URL과 HTTP 메소드를 꺼내서 HandlerKey를 만든다.
      - HandlerExecution
        - Object, Method
        - handle() 메소드를 호출하면 메소드를 실행시켜 준다.

  - ServletContext에 DispatcherServlet 등록

- DispatcherServlet

  - 요청이 들어오면 service() 메소드가 호출된다.
  - getHandler()
    - HandlerExecution 리턴
  - execute() 
    - 여러 개의 프레임워크 컨트롤러를 추상화한 HandlerAdapter 타입이 HandlerExecution을 실행 시킨다.



### 10.3 인터페이스가 다른 경우 확장성 있는 설계





### 10.4 배포 자동화를 위한 쉘 스크립트 개선

## 11장 의존관계 주입(이하 DI)을 통한 테스트하기 쉬운 코드 만들기

### 11.1 왜 DI가 필요한가?

### 11.2 DI를 적용하면서 쌓이는 불편함(불만)

### 11.3 불만 해소하기

### 11.4 DI 프레임워크 실습

### 11.5 DI 프레임워크 구현

### 11.6 추가 학습 자료

## 12장 확장성 있는 DI 프레임워크로 개선

### 12.1 필드와 setter 메소드에 @Inject 기능 추가

### 12.2 필드와 setter 메소드 @Inject 구현

### 12.3 @Inject 개선

### 12.4 설정 추가를 통한 유연성 확보

### 12.5 외부 라이브러리 클래스를 빈으로 등록하기

### 12.6 초기화 기능 추가

### 12.7 인터페이스, DI, DI 컨테이너

### 12.8 웹 서버 도입을 통한 서비스 운영

## 13장 세 번째 양파 껍질 벗기기

### 13.1 스프링과 ORM 프레임워크

### 13.2 성능과 보안

### 13.3 프론트엔드 학습

### 13.4 설계, 테스트, 리팩토링

### 13.5 빌드, 배포 자동화 및 지속적 통합

### 13.6 개발 문화 및 프로세스 학습