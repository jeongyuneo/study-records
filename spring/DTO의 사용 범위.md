# [Spring Boot] DTO의 사용 범위
## DTO란
DTO(Data Transfer Object)란 계층간 데이터 교환을 위해 사용하는 객체(Java Beans)이다. DTO를 설명하기 위해선 먼저 MVC 패턴에 대해 알아야 한다.

### MVC 패턴

![웹 애플리케이션에서 일반적인 MVC 구성요소 다이어그램](https://images.velog.io/images/minide/post/29966859-4aca-4b35-b238-f9aabf5dfe68/300px-Router-MVC-DB.svg.png)

MVC 패턴(Model-View-Controller Pattern)은 애플리케이션 개발 시 그 구성요소를 Model과 View, Controller 세 가지 역할로 구분하는 디자인 패턴이다. 비즈니스 처리 로직(Model)과 UI 영역(View)의 중간에서 Controller가 연결해주는 역할을 한다.

**Controller**는 **View**로부터 들어온 사용자 요청을 해석해 **Model**을 업데이트하거나 **Model**로부터 데이터를 받아 **View**로 전달하는 작업 등을 수행한다. MVC 패턴의 장점은 Model과 View를 분리함으로써 서로의 **의존성을 낮추고** 독립적인 개발을 가능하게 한다.

Controller는 View와 도메인 Model의 데이터를 주고 받을 때 별도의 DTO를 주로 사용한다. 도메인 객체를 View에 직접 전달할 수 있지만, 도메인의 비즈니스 기능이 노출될 수 있으며 Model과 View 사이에 의존성이 생기기 때문이다.

### DTO를 사용하지 않는 경우

| User.java
```java
public class User {

  private Long id;
  private Sting name;
  private String email;
  private String password;
  ...
}
```
| UserController.java
```java
@GetMapping("/page/{id}")
@ResponseStatus(HttpStatus.OK)
public User viewMyPage(@PathVariable("id") Long id) {
    return userService.viewMyPage(id);
}
```

이렇게 Controller가 클라이언트의 요청에 대한 응답으로 도메인 Model인 User를 넘겨주면 다음와 같은 문제점이 있다.

1. 도메인 Model의 모든 속성이 외부에 노출된다.
    - 위 예시의 경우 사용자가 마이페이지 조회에 대한 요청을 하는데 그에 대한 응답으로 외부에 노출되면 안되는 'password' 데이터까지 응답해준다. 이처럼 도메인 Model 자체를 응답해주면 중요한 정보가 외부에 노출되는 보안 문제가 발생한다.
    - 또한 View에서 보낸 요청에 대한 데이터 외의 불필요한 데이터를 모두 가지고 있다.
2. UI 계층에서 Model의 메소드를 호출하거나 상태를 변경시킬 위험이 존재한다.
3. Model과 View가 강하게 결합되어, View의 요구사항 변화가 Model에 영향을 끼치기 쉬워진다.
    - 또한 User Entity의 속성이 변경되면, View가 전달받을 JSON 등 클라이언트의 코드에도 변경을 유발하기 때문에 상호간 강하게 결합된다.

### DTO를 사용하는 경우

| UserResponseDto.java
```java
public class UserResponseDto {

  private String name;
  private String email;
}
```

| UserController.java
```java
@GetMapping("/page/{id}")
@ResponseStatus(HttpStatus.OK)
public UserResponseDto viewMyPage(@PathVariable("id") Long id) {
    return userService.viewMyPage(id);
}
```

DTO를 사용하면 앞에서 언급했던 문제들을 해결할 수 있다. 도메인 Model을 캡슐화하고, UI 화면에서 사용하는 데이터만 선택적으로 보낼 수 있다.

> 정리하면 DTO는 클라이언트 요청에 포함된 데이터를 담아 서버 측에 전달하고, 서버 측의 응답 데이터를 담아 클라이언트에 전달하는 계층간 전달자 역할이다.

## DTO의 사용 범위
### Controller가 Service단으로 DTO를 넘겨주는 경우
| UserController.java
```java
@PostMapping("/login")
@ResponseStatus(HttpStatus.OK)
public LoginResponseDto login(@RequestBody LoginRequestDto loginRequestDto) {
    return userService.login(loginRequestDto);
}
```
| UserService.java
```java
public LoginResponseDto login(LoginRequestDto loginRequestDto) {
    User findUser = userRepository.findByEmail(loginRequestDto.getEmail())
                .orElseThrow(() -> new NotFoundException(USER_NOT_FOUND_MESSAGE));
    ...
    return new LoginResponseDto(findUser.getId(), new Message(LOGIN_SUCCESS_MESSAGE));
}
```
위 코드는 현재 진행 중인 토이 프로젝트 코드의 일부다. 평소 습관대로 프로그래밍을 하다가 문득 **'Service 단에 Dto를 넘겨주는게 아니라 안에서 필요한 값들만 넘겨줘야 하는거 아닌가?'** 다시 말해, **'DTO와 Domain 간의 변환 위치가 꼭 Controller여야 할까?'** 라는 의문이 들었다.

### Controller가 Service단으로 Domain 객체를 넘겨주는 경우
| UserController.java
```java
@PostMapping("/login")
@ResponseStatus(HttpStatus.OK)
public LoginResponseDto login(@RequestBody LoginRequestDto loginRequestDto) {
    User user = userService.login(loginRequestDto.toEntity());
    return new LoginResponseDto(user);
}
```
이처럼 Controller단에서 Domain으로 변환해서 Service단에 넘겨줘도 정상적으로 동작한다. 마찬가지로, Service단에서 Dto가 아닌 Domain 객체를 Controller단에 넘겨주면 Controller단에서 Dto를 생성해 반환해주어도 정상적으로 동작한다.

하지만 Controller단에서 Dto를 처리해주면 위에 예시처럼 아주 간단한 응답 객체를 리턴받는 경우라면 상관이 없지만 로직이 좀 더 복잡해지면 Controller단의 코드가 복잡해지게 된다. 즉, **Controller가 여러 Service 객체에 의존하게 된다.**

또한 Controller와 Service Layer가 DTO가 아닌 Domain 객체를 넘기게 되면 **불필요한 필드를 갖게 된다.**

### Repository에서 DTO를 반환해주면 안될까?
결론부터 말하자면, 이 방법은 지양해야 한다.

Repository는 Aggregate의 영속성과 Repository 자체만을 고려해야한다. 따라서 Repository의 책임은 Presentation Layer와 Aggregate의 상태 공유가 아니라, Aggregate의 상태를 영속하는 것에 있기 때문이다.

## 그렇다면 어떤 방법이 맞는 것인가?
명확하게 어떤 방법이 맞다고 결론 짓기 어려운 문제 같다.

지금은 개인적으로 Service Layer에서 DTO의 변환을 처리해야 한다고 생각한다. Service Layer가 Domain과 Controller를 연결해주는 매체라고 생각하기 때문에 Domain에서 사용하고자 하는 필드를 Controller에 전달하기 위한 객체인 DTO를 처리하는 것도 Domain과 Controller 사이에서 즉, Service Layer에서 진행돼야 한다고 생각한다.

## 마치며
개인적으로 Service Layer에서 DTO의 변환을 처리해야 한다고 생각했다. Service Layer가 Domain과 Controller를 연결해주는 매체이기 때문에 Domain에서 사용하고자 하는 필드만을 Controller에 전달하기 위한 객체인 DTO를 처리하는 것도 Domain과 Controller 사이에서 즉, Service Layer에서 진행돼야 한다고 생각했다.

하지만 [DTO의 사용 범위에 대하여](https://tecoble.techcourse.co.kr/post/2021-04-25-dto-layer-scope/) 글을 읽어보며 마지막에 첨부된 피드백을 보고 Controller 단에서 DTO의 변환을 처리해주는 방법에 대해 다시 한번 생각해봤다.

따라서 한 가지 방법이 맞다고 결론을 정해놓고 사용하기 보다는 상황에 맞게 필요한 방법을 선택해서 사용하는 것이 좋은 방법이라고 생각한다.

## 참고
[Entity To DTO Conversion for a Spring REST API](https://www.baeldung.com/entity-to-and-from-dto-for-a-java-spring-application)

[A Better Way To Project Domain Entities into DTOs](https://buildplease.com/pages/repositories-dto/)

[Should services always return DTOs, or can they also return domain models?](https://stackoverflow.com/questions/21554977/should-services-always-return-dtos-or-can-they-also-return-domain-models)

[DTO의 사용 범위에 대하여](https://tecoble.techcourse.co.kr/post/2021-04-25-dto-layer-scope/)

[DTO는 어느 레이어까지 사용하는 것이 맞을까?](https://www.slipp.net/questions/93)