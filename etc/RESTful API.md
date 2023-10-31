# RESTful API

## REST

REST란, REpresentational State Transfer의 약자이다. 여기에 ~ful 이라는 형용사형 어미를 붙여 ~한 API 라는 표현으로 사용된다. 즉, REST의 기본 원칙을 성실히 지킨 서비스 디자인은 'RESTful'하다고 표현할 수 있다.

REST는 하나의 아키텍처로 볼 수 있다. 좀 더 정확한 표현으로 말하자면, REST 는 **Resource Oriented Architecture**이다. API 설계의 중심에 자원(Resource)이 있고 HTTP Method를 통해 자원을 처리하도록 설계하는 것이다.

### REST 특징

- **Server-Client**: 자원을 요청하는 클라이언트와 자원을 처리하는 서버로 구성된다.
- **Stateless**: 클라이언트와 서버의 통신에는 상태가 없어야 한다. 즉, 모든 요청은 필요한 모든 정보를 담고 있어야 한다.
- **Uniform Interface**: 구성 요소 간 인터페이스는 균일해야 한다. 즉, 특정 언어나 기술에 종속되지 않는다.
- **Cachable**: 캐시가 가능해야 한다.
- **Layered System**: 계층으로 구성이 가능해야 하며, 각 레이어에 속한 구성 요소는 인접하지 않은 레이어의 구성요소를 볼 수 없어야 한다.
- Code-On-Demand(Optional): 서버가 네트워크를 통해 클라리언트에 프로그램을 전달하면 그 프로그램이 클라이언트에서 실행될 수 있어야 한다.

## RESTful API

1. **리소스**와 **행위**를 명시적이고 직관적으로 분리한다.
    - 리소스는 **URI**로 표현되는데 리소스가 가리키는 것은 **명사**로 표현되어야 한다.
    - 행위는 **HTTP Method**로 표현하고, `GET(조회)`, `POST(생성)`, `PUT(기존 entity 전체 수정)`, `PATCH(기존 entity 일부 수정)`, `DELETE(삭제)`을 분명한 목적으로 사용한다.
2. 메시지는 **헤더(Header)와 바디(Body)를 명확하게 분리**해서 사용한다.
    - 엔티티에 대한 내용은 바디에 담는다.
    - 애플리케이션 서버가 행동할 판단의 근거가 되는 컨트롤 정보인 API 버전 정보, 응답받고자 하는 MIME 타입 등은 헤더에 담는다.
    - 헤더와 바디는 **HTTP Header**와 **HTTP Body**로 나눌 수도 있고, HTTP Body에 들어가는 JSON 구조로 분리할 수도 있다.
3. API **버전을 관리**한다.
    - 환경은 항상 변하기 때문에 API의 signature가 변경될 수도 있음에 유의하자.
    - 특정 API를 변경할 때는 반드시 하위호환성을 보장해야 한다.
4. **서버와 클라이언트가 같은 방식을 사용**해서 요청하도록 한다.
    - 브라우저는 `form-data` 형식의 submit으로 보내고 서버에서는 JSON 형태로 보내는 식의 분리보다는 JSON으로 보내든, 둘 다 `form-data` 형식으로 보내든 하나로 통일한다.
    - 다른 말로 표현하자면 URI가 **플랫폼 중립적**이어야 한다.

## References

[RESTful API란](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/main/Development_common_sense#restful-api)

[바쁜 개발자들을 위한 REST 논문 요약](https://blog.npcode.com/2017/03/02/%EB%B0%94%EC%81%9C-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%93%A4%EC%9D%84-%EC%9C%84%ED%95%9C-rest-%EB%85%BC%EB%AC%B8-%EC%9A%94%EC%95%BD/)