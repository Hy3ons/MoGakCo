---
title: 하계 모각코 02
description: 2025년-하계 두번째 모각코 게시글
slug: 2025-01-mogakco-2
date: 2025-07-27 21:20:00+0900
image: cover.png
author: "황현석"
categories:
    - spring
weight: 1
---

# 하계 모각코 02 - 목표

이번 모각코 시간에는, udemy spring 강의의 143 번째 강의부터 150분간 강의를 듣고 공부한 내용과 궁금한 부분 따로 알아분 부분들에 대해서 추가적으로 적어볼 예정입니다.

대체로 강의 내용은, JPA와 Hibernates 를 다루며, REST API를 만들어 보는 영상이며, 추가로 에러 핸들링과, Spring Security등이 있습니다.

## 오늘의 목표 강의 목록

1. 모든 리소스에 대한 예외 처리 구현
2. DELETE 메소드로 사용자 리소스 삭제
3. REST API에서 유효성 검증 적용
4. 고급 REST API 기능 개요 파악
5. Open API 사양 및 Swagger 이해
6. springdoc-openapi 의존성 추가 및 Swagger 문서 자동 생성 구성
7. 콘텐츠 협상 및 XML 지원 구현
8. REST API의 국제화(i18n) 적용
9. REST API 버전 관리 (URI, 요청 매개변수, 헤더, 콘텐츠 협상 방식)

짧은 시간 동안 여러 주제를 다루겠지만, 각 단계별로 핵심만 정리해보는 것이 오늘의 목표입니다.


# 하계 모각코 02 - 결과

## 전역 예외 처리 - @ControllerAdvice + ExceptionHandler

```java
@ControllerAdvice
public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(Exception.class)
    public final ResponseEntity<Object> handleAllException(Exception ex, WebRequest request) {
        ErrorDetails errorDetails = new ErrorDetails(LocalDateTime.now(), ex.getMessage(), request.getDescription(false));
        return new ResponseEntity<>(errorDetails, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

예외에 관한 내용이다. 이런식으로, @ExceptionHandler와 관련된 @ControllerAdvice 애너테이션으로 Bean을 등록해주고, 상속받은 메서드와 새롭게 정의된 내부 ExceptionHandler를 통해 에러를 다룰 수 있다.

ResponseEntityExceptionHandler를 상속하는 이유는 다음과 같다.  
@ControllerAdvice는 Spring의 전역 예외 처리자이다. 그래서 상속하지 않으면, @Valid, @RequestBody 등의 에러는 InternalServerError로 처리될 수 있다.

ResponseEntityExceptionHandler에 기본적으로 정의되어 있는 메서드들,  
handleMethodArgumentNotValid나, handleHttpMessageNotReadable 같은 기본적인 오류 처리 로직을 사용할 수 있다.  

우리가 전역 예외 처리자를 새롭게 등록하면, Spring에서는 기본 예외 처리 로직을 지원 중단하기 때문에, 기존 것을 모두 사용하기 위해서는 **상속을 받아야** 한다.

저런식으로 리턴을 하면, errorDetails라는 객체가 JSON, stringify 되어 response로 날아간다.  
예외를 쉽게 client에 전역적으로 처리할 수 있게 되는 셈이다.

---

## 유효성 검증 - @Valid + Jakarta Validation

각 파라미터의 유효성 검증은 파라미터 앞에 @Valid 애너테이션을 다는 것으로 배웠었다.  
하지만 이것은 @ResponseBody로 객체 타입이 파라미터로 주어져도 @Valid로 유효성 검증을 수행할 수 있다.

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
    User savedUser = userRepository.save(user);
    return ResponseEntity.ok(savedUser);
}
```

이런식으로 @Valid를 넣어주고,

```java
@Entity(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;

    @Size(min = 2, message = "이름은 적어도 2글자 이상 넣어주세요.")
    private String name;

    @Past(message = "미래는 생일로 지정 할 수 없습니다.")
    private LocalDate birthdate;

    private Integer solver;
}
```

검증할 각 객체의 멤버에 Jakarta 형식의 검증 애너테이션을 넣으면 된다.  
전역적으로 @Valid 오류는 그 원인을 클라이언트에게 기본적으로 전달하지 않는다.

```java
@Override
protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex,
                                                              HttpHeaders headers,
                                                              HttpStatusCode status,
                                                              WebRequest request) {
    ErrorDetails er = new ErrorDetails(LocalDateTime.now(), ex.getMessage(), request.getDescription(false));
    return new ResponseEntity&lt;&gt;(er, HttpStatus.BAD_REQUEST);
}
```

이런식으로 ResponseEntityExceptionHandler의 메서드를 오버라이딩하여 에러 처리 시 client 에게 돌아가는 내용을 수정할 수 있다.

```java
@Override
protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex,
                                                              HttpHeaders headers,
                                                              HttpStatusCode status,
                                                              WebRequest request) {
    ErrorDetails er = new ErrorDetails(LocalDateTime.now(),
            ex.getFieldErrors().stream()
              .map(fieldError -> fieldError.getDefaultMessage())
              .collect(Collectors.joining(", ")),
            request.getDescription(false));
    return new ResponseEntity<>(er, HttpStatus.BAD_REQUEST);
}
```

---

## Swagger를 이용한 문서 자동화

Swagger 적용은 매우 간단했다. 의존성 추가만으로 적용되며, input schema를 대충 지정해도 알아서 잘 매핑해준다.

```xml
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

이러한 종속성 추가를 통해, client의 header에 "accept" : "application/xml"을 넣으면  
자동으로 반환하는 콘텐츠의 반환 형식을 제공할 수 있다.  
Swagger와 XML은 문서 충돌을 일으킬 수 있으니 주의하자.

---

## 국제화 (i18n) 지원

resources 폴더에 messages_{tag}.properties 파일들을 만들어 다국어 처리를 할 수 있다.

```java
@GetMapping(path="/hello-world-i18n")
public String hel() {
    Locale locale = LocaleContextHolder.getLocale();
    return messageSource.getMessage("good.morrning.message", null, "default", locale);
}
```

`LocaleContextHolder.getLocale()`을 통해 현재 요청의 locale을 추출하고,  
그에 맞는 메시지를 리턴해준다.

---

## API 버전 전략

### URI 버전 방식 (Twitter 스타일)
```
GET /v1/users
GET /v2/users
```

### 요청 파라미터 방식 (Amazon 스타일)
```
GET /users?version=1
GET /users?version=2
```

### Header 방식 (Microsoft 스타일)
```
Header: API_VERSION=1
Header: API_VERSION=2
```

API의 request/response DTO에 변화가 생기더라도 기존 consumer들을 보호할 수 있는 좋은 전략이다.