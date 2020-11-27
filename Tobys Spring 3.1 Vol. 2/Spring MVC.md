

## 4.1 @REQUESTMAPPING 핸들러 매핑



## 4.2 @Controller



## 4.3 모델 바인딩과 검증



## 4.4 JSP 뷰와 form 태그



## 4.5 메시지 컨버터와 AJAX



## 4.6 MVC 네임스페이스



## 4.7 @MVC 확장 포인트



## 4.8 URL과 리소스 관리



## 4.9 스프링 3.1의 @MVC



### 4.9.3 @RequestMapping 핸들러 어댑터

- HandlerMethod 타입의 핸들러를 담당하는 핸들러 어댑터는 RequestMappingHandlerAdapter다.
- RedirectAttributes와 리다이렉트 뷰
  - 리다이렉트는 요청에 대한 응답으로 뷰를 보내는게 아니라 브라우저에게 새로운 URL로 요청을 다시 보내라고 지시하는 응답 방식
    - 브라우저에 마지막 전송 결과가 남아 있다면 바로 응답으로 뷰를 보내서 페이지 갱신하는 경우에는 폼이 다시 전송될 위험이 있다.
  - 리다이렉트 뷰를 위해 모델 정보를 사용할 때는 Model 대신 RedirecAttributes를 사용하면 편리하다.
- 4.7절에 소개한 WebArgumentResolver는 스프링 3.0의 @MVC에서 지원하는 핸들러 메소드의 파라미터 타입 확장 포인트다.
  - 스프링 3.1에서는 메소드 호출과 모델 바인딩, 파라미터 변환 등을 담당하는 핸들어 어댑터가 바뀌었기 때문에 확장 포인트도 달라진다.

- 파라미터: HandlerMethodArgumentResolver
  - 핸들러 메소드에서 사용할 수 있는 새로운 종류의 파라미터 타입을 추가하려면?
    - HandlerMethodArgumentResolver 인터페이스를 구현
    - 구현한 걸 RequestMappingHandlerAdapter의 customArgumentResolvers 프로퍼티에 추가
- 리턴 값: HandlerMethodReturnValueHandler
  - 리턴 값을 처리하는 기능도 확장할 수 있다.
  - HandlerMethodReturnValueHandler 인터페이스를 구현
  - 구현한 걸 RequestMappingHandlerAdapter의 customReturnValueHandlers 프로터티에 추가

- 이 두 가지 확장 포인트는 RequestMappingHandlerAdapter가 기본적으로 제공하는 파라미터와 리턴 값 처리 클래스에서도 사용된다.



## 4.10 정리



