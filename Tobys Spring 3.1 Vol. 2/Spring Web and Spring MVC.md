

- 웹 프레젠테이션 계층
  - 엔터프라이즈 애플리케이션의 가장 앞단에서 사용자 또는 클라이언트 시스템과 연동하는 책임을 맡고 있다.
  - 3 Tier
    - 프레젠테이션 계층
    - 서비스 계층
    - 데이터 엑세스 계층



## 3.1 스프링의 웹 프레젠테이션 계층 기술

- 프레젠테이션 계층은 복잡하고 다양한 기술의 조합으로 구성 될 수 있다.
- 스프링 애플리케이션 입장에서는 두 가지로 구분
  - 스프링 웹 기술을 사용하는 프레젠테이션 계층
  - 스프링 외의 웹 기술을 사용하는 프레젠테이션 계층
- 스프링은 의도적으로 서블릿 웹 어플리케이션의 컨텍스트를 두 가지로 분리함.
  - 루트 애플리케이션 컨텍스트
    - 웹 기술에서 완전히 독립적인 비즈니스 서비스 계층과 데이터 엑세스 계층을 담고 있다.
  - 서블릿 애플리케이션 컨텍스트
    - 스프링 웹 기술을 기반으로 동작하는 웹 관련 빈을 담고 있다.
- 스프링 컨텍스트를 두 가지로 분리해둔 이유는 스프링 웹 서블릿 컨텍스트를 통째로 다른 기술로 대체할 수 있도록 하기 위해서다.



### 3.1.1 스프링에서 사용되는 웹 프레임워크의 종류





스프링 웹 프레임워크
스프링 포트폴리오 웹 프레임워크
스프링을 기반으로 두지 않는 웹 프레임워크





### 3.1.2 스프링 MVC와 DispatcherServlet 전략

- DispatcherServlet과 MVC 아키텍처
  - (1) DispatcherServlet의 HTTP 요청 접수
  - (2) DispatcherServlet에서 컨트롤러로 HTTP 요청 위임
    - DispatcherServlet은 URL, 파라미터 정보, HTTP 명령 등을 참고해 어떤 컨트롤러에 작업을 위임할지 결정한다.
    - 컨트롤러를 선정하는 것은 DispatcherServlet의 핸들러 매핑 전략을 이용한다.
      - 스프링에서는 **컨트롤러를 핸들러라고** 부른다.
        - 웹의 요청을 다루는 핸들하는 오브젝트라는 의미다.
    - DispatcherServlet이 매핑으로 찾은 컨트롤러를 가져와 실행하려면 컨트롤러 메소드를 어떻게 호출할지를 알고 있어야 한다.
      - 어떻게 알 수 있을까? 어떻게 자바의 오브젝트 사이에 요청이 전달될 수 있을까?
        - 컨트롤러에는 특정 인터페이스를 구현해야 한다던지 하는 제약이나 선결 조건이 없다.
          - 근데 어떻게 알 수 있을까나
        - 해결책은 **어댑터를 이용**하는 것이다.
          - 전형적인 오브젝트 어댑터 패턴을 사용
          - 특정 컨트롤러를 호출해야 할 때는 컨트롤러 타입을 지원하는 어댑터를 중간에 껴서 호출
          - 오케이 이제 **핸들러 어댑터**라는 용어가 자연스럽게 이해가 된다.



- DispatcherServlet의 DI 가능한 전략




## 3.2 스프링 웹 애플리케이션 환경 구성

3.2.1 간단한 스프링 웹 프로젝트 생성
루트 웹 애플리케이션 컨텍스트
서블릿 웹 애플리케이션 컨텍스트 등록
스프링 웹 프로젝트 검증
3.2.2 스프링 웹 학습 테스트
서블릿 테스트용 목 오브젝트
테스트를 위한 DispatcherServlet 확장
ConfigurableDispatcherServlet을 이용한 스프링 MVC 테스트
편리한 DispatcherServlet 테스트를 위한 AbstractDispatcherServletTest



## 3.3 컨트롤러



3.3.1 컨트롤러의 종류와 핸들러 어댑터
Servlet과 SimpleServletHandlerAdapter
HttpRequestHandler와 HttpRequestHandlerAdapter
Controller와 SimpleControllerHandlerAdapter
AnnotationMethodHandlerAdapter
3.3.2 핸들러 매핑
BeanNameUrlHandlerMapping
ControllerBeanNameHandlerMapping
ControllerClassNameHandlerMapping
SimpleUrlHandlerMapping
DefaultAnnotationHandlerMapping
기타 공통 설정정보
3.3.3 핸들러 인터셉터
HandlerInterceptor
핸들러 인터셉터 적용
3.3.4 컨트롤러 확장
커스텀 컨트롤러 인터페이스와 핸들러 어댑터 개발


## 3.4 뷰



3.4.1 뷰
InternalResourceView와 JstlView
RedirectView
VelocityView, FreeMarkerView
MarshallingView
AbstractExcelView, AbstractJExcelView, AbstractPdfView
AbstractAtomFeedView, AbstractRssFeedView
XsltView, TilesView, AbstractJasperReportsView
MappingJacksonJsonView
3.4.2 뷰 리졸버
InternalResourceViewResolver
VelocityViewResolver, FreeMarkerViewResolver
ResourceBundleViewResolver, XmlViewResolver, BeanNameViewResolver
ContentNegotiatingViewResolver


## 3.5 기타 전략



3.5.1 핸들러 예외 리졸버
AnnotationMethodHandlerExceptionResolver
ResponseStatusExceptionResolver
DefaultHandlerExceptionResolver
SimpleMappingExceptionResolver
3.5.2 지역정보 리졸버
3.5.3 멀티파트 리졸버

- 멀티 파트 처리를 담당하는 다양한 구현으로 바꿀 수 있도록 설계되어 있다.
  - 현재는 아파치 `Commons`의 `FileUpload` 라이브러리를 사용하는 `CommonsMultipartResolver` 한 가지만 지원된다.
- 멀티파트 리졸버 전략은 디폴트로 등록되는 것이 없다.
  - 스프링부트에서는 `MultipartAutoConfiguration` 클래스 에서 `StandardServletMultipartResolver` 클래스를 등록해준다. 
- DispatcherServlet이 클라이언트로부터 멀티파트 요청을 받으면 멀티파트 리졸버에게 요청
  - 멀티파트 리졸버가 HttpServletRequest의 확장 타입인 MultipartHttpServletRequest 오브젝트로 전환
    - 멀티파트 리졸버가 멀티파트 문제를 리졸브 해준다(해결해주다).



RequestToViewNameTranslator



## 3.6 스프링 3.1의 MVC



3.6.1 플래시 맵 매니저 전략
플래시 맵
플래시 맵 매니저
플래시 맵 매니저 전략
3.6.2 WebApplicationInitializer를 이용한 컨텍스트 등록
루트 웹 컨텍스트 등록
서블릿 컨텍스트 등록



## 3.7 정리

- 웹 프레젠테이션 계층?



