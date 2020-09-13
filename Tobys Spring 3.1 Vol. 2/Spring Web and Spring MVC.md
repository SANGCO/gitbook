



## 3장 스프링 웹 기술과 스프링 MVC

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



DispatcherServlet과 MVC 아키텍처
DispatcherServlet의 DI 가능한 전략




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



