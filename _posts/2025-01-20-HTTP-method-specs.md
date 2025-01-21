---
title: "HTTP 메소드 스펙"
layout: post
categories: development
tags:

- web
---

## 들어가며

이 글은 기본적으로 [httpwg.org/specs](https://httpwg.org/specs), 특히 HTTP Semantics[1]의 Methods 섹션과 PATCH Method[2] 문서를 기반으로 작성했다. 캐싱은 집중적으로 다루지 않는다. 

RESTful API에서 사용할 HTTP 메소드를 선택할 때 도움이 될만한 정보들을 정리했다. HTTP 스펙을 엄격히 지키는 프로젝트는 별로 없는 것 같으니 참고 정도만 하면 될 것 같다. 

 

## 메소드 공통 성질

GET, POST 등을 메소드 토큰이라 부른다. 이는 **의미적으로** 클라이언트가 요청을 만든 목적과 기대하는 성공 결과를 나타낸다. 조건부 요청 같은 몇몇 헤더는 요청의 의미를 더욱 구체화할 수 있다. 토큰은 대소문자를 구분하고, 관례에 따라 US-ASCII 대문자로 정의된다. 

메소드에 공통적으로 적용되는 성질로 (1) Safe, (2) Idempotent, (3) Caching 이 있다. 

(1) 어떤 메소드의 의미가 본질적으로 Read-Only일 경우, 해당 메소드를 **Safe**하다고 말한다. GET, HEAD, OPTIONS, TRACE 메소드가 해당된다. 클라이언트는 Safe 메소드를 요청함으로써 원본 서버의 상태가 변경되기를 기대하지 않는다. 이는 어디까지나 요청된 메소드의 본질적인 **의미**에 관한 것으로, 실제 서버의 구현이 side effect가 전혀 없는 완전한 read only임을 보장하진 않는다. 서버가 GET 메소드에 대해서 로깅하더라도, 여전히 Safe하다고 간주된다. 

(2) 어떤 요청을 1번 or N번 보냈을 때, 둘이 의도한 바(서버에 미치는 영향)가 같다면 해당 메소드를 **Idempotent**하다고 말한다. PUT, DELETE 및 Safe 메소드가 해당된다. 통신이 실패했을 때, 클라이언트는 서버의 응답을 받기 전에 Idempotent 메소드를 자동으로 재시도할 수 있지만, 원래 요청과 재시도 요청의 응답 결과는 달라질 수 있다. 클라이언트는 실패한 자동 재시도 요청을 자동으로 재시도해서는 안된다. non-Idempotent 메소드 요청은 자동으로 재시도해서는 안되지만, 원래 요청이 전혀 적용되지 않았음을 감지할 수 있는 경우는 예외이다. 

(3) 캐싱을 위해서는 관련 메소드가 명시적으로 캐싱을 허용해야 하며, 이후 요청에서 응답이 재사용될 수 있는 조건을 제공해야 한다. 스펙상으로는 GET, HEAD, POST의 캐싱이 정의되어 있지만, 대부분의 구현체는 GET과 HEAD만을 지원한다. 



다음은 표준 HTTP 메소드의 표이다. PATCH 메소드는 나중에 추가되었기 때문에, PUT 메소드와 유사하게 설명을 추가했다. 

| Method Name | Description                                                  | Section                                                |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| GET         | Transfer a current representation of the target resource.    | [9.3.1](https://httpwg.org/specs/rfc9110.html#GET)     |
| HEAD        | Same as GET, but do not transfer the response content.       | [9.3.2](https://httpwg.org/specs/rfc9110.html#HEAD)    |
| POST        | Perform resource-specific processing on the request content. | [9.3.3](https://httpwg.org/specs/rfc9110.html#POST)    |
| PUT         | Replace all current representations of the target resource with the request content. | [9.3.4](https://httpwg.org/specs/rfc9110.html#PUT)     |
| DELETE      | Remove all current representations of the target resource.   | [9.3.5](https://httpwg.org/specs/rfc9110.html#DELETE)  |
| CONNECT     | Establish a tunnel to the server identified by the target resource. | [9.3.6](https://httpwg.org/specs/rfc9110.html#CONNECT) |
| OPTIONS     | Describe the communication options for the target resource.  | [9.3.7](https://httpwg.org/specs/rfc9110.html#OPTIONS) |
| TRACE       | Perform a message loop-back test along the path to the target resource. | [9.3.8](https://httpwg.org/specs/rfc9110.html#TRACE)   |
| PATCH       | Replace partial current representations of the target resource with request content. | [2](https://httpwg.org/specs/rfc5789.html#patch)       |

출처: <https://httpwg.org/specs/rfc9110.html#method.overview>



## 표준 메소드 정의 (GET, HEAD, POST, PUT, DELETE, PATCH)

### GET

클라이언트는 요청에 `Range` 헤더 필드를 포함하여, 선택된 표현의 일부만 전송하도록 요청할 수 있다. 

GET 요청의 콘텐츠(Body)는 일반적으로 정의된 의미가 없고, 요청의 의미나 대상을 변경할 수 없다. 클라이언트는 GET 요청에 콘텐츠를 생성하지 않아야 한다. 일부 구현에서는 smuggling attack 예방을 위해 그러한 요청을 거부할 수 있다. 예외적으로, 서버가 그러한 요청이 목적이 있고 지원될 것임을 명시한 경우에는 가능하다. 

GET 요청에 대한 응답은 캐시 가능하다. `Cache-Control` 헤더 필드에 의해 달리 지정되지 않는 한, 후속 GET 및 HEAD 요청을 충족시키는 데 사용할 수 있다. 

URI 내에 민감한 정보가 포함될 경우, 필터링되거나 변환될 수 있다. 캐싱에 이점이 없을 경우, POST 메소드를 사용할 수도 있다. 

### HEAD

HEAD 메소드는 GET과 동일하지만, 서버는 응답에 **콘텐츠를 포함하지 않아야** 한다. 

GET 요청과 마찬가지로, HEAD 요청의 콘텐츠는 일반적으로 정의된 의미가 없다. 클라이언트는 서버에 의해 명시된 경우를 제외하고는 HEAD 요청에 콘텐츠를 포함하면 안 된다. 

HEAD 응답은 이전에 캐시된 GET 응답을 무효화하거나 업데이트하는 데 사용될 수 있다. 

### POST

POST 메소드는 요청에 포함된 representation을 대상 리소스가 자체적인 의미론에 따라 처리한다. 대부분의 일을 수행할 수 있다. 대표적인 경우는 다음과 같다:

- Providing a block of data, such as the fields entered into an HTML form, to a data-handling process;
- Posting a message to a bulletin board, newsgroup, mailing list, blog, or similar group of articles;
- **Creating a new resource** that has **yet to be identified** by the origin server; and
- Appending data to a resource's existing representation(s).

서버는 POST 요청 결과에 따라 적절한 상태 코드를 선택하여 응답의 의미를 나타낸다. 206(Partial Content), 304(Not Modified), 416(Range Not Satisfiable)을 제외한 모든 상태 코드가 POST 응답으로 사용될 수 있다. 

하나 이상의 리소스가 생성된 경우, 응답은 201(Created) 상태 코드, Location 헤더 필드(생성된 리소스의 식별자), 새 리소스 상태를 설명하는 representation을 포함해야 한다. 

특정 조건을 만족하는 경우 POST 응답은 캐시될 수 있다. 해당 캐시는 이후의 GET, HEAD 요청에서 사용될 수 있지만, POST 요청에서는 사용될 수 없다. POST는 Safe하지 않기 때문이다. 

### PUT

PUT 메소드는 요청에 포함된 representation을 바탕으로 대상 리소스의 상태를 생성하거나 대체하도록 요청한다. 이후에 발생하는 해당 리소스에 대한 GET 요청은 200(OK)와 PUT 요청에 포함된 것과 동등한 representation을 반환할 것이다. 

서버는 요청에 포함된 representation이 대상 리소스의 제약 조건을 만족하는지 확인해야 한다. 만족하지 않는 경우, (1) 요청된 representation 또는 대상 리소스의 구성을 변경하여 조건을 만족시키거나, (2) 적절한 오류 메시지(409 Conflict, 415 Unsupported Media Type)와 함께 요청을 거부해야 한다. 

대상 리소스에 현재 representation이 없었고 성공적으로 생성한 경우 201(Created)를 반환한다. 기존 representation이 있었고 성공적으로 수정한 경우 200(OK) or 204(No Content)를 반환한다. 

PUT 요청의 올바른 해석은, 사용자 에이전트가 원하는 대상 리소스를 명확히 알고 있다는 전제를 갖는다. 클라이언트의 상태 변경 요청 후에, 서버가 그에 대한 적절한 URI를 선택하는 경우에는 PUT이 아닌 POST 메소드로 구현돼야 한다. 실제 REST 시나리오에서 클라이언트는 보통 서버가 제공한 URI를 사용하므로, 리소스 생성에 PUT 메소드를 사용하는 것은 기본적으로 권장되지 않는다. [3]

PUT 요청에 대한 응답은 캐시 불가능하다. 성공적인 PUT 요청이 캐시를 통과하면, 해당 대상 URI에 대해 저장된 응답 캐시들은 모두 무효화된다. 

### DELETE

DELETE 메소드는 서버가 대상 리소스와 현재 기능 간의 연관 관계를 제거하도록 요청한다. UNIX의 rm 명령어와 비슷하다: 이전에 연관되었던 정보가 삭제되는 것이라기보다, URI 매핑을 삭제하는 것을 표현한다. 

GET, HEAD 요청과 마찬가지로, DELETE 요청의 콘텐츠는 일반적으로 정의된 의미가 없다. 클라이언트는 서버에 의해 명시된 경우를 제외하고는 DELETE 요청에 콘텐츠를 포함하면 안 된다. 

DELETE 메소드가 성공적으로 적용된 경우, 다음 상태 코드가 반환된다:

- 202 Accepted: 작업이 성공할 가능성이 높지만 아직 실행되지 않았다.
- 204 No Content: 작업이 실행되었으며 추가 정보를 제공할 필요가 없다.
- 200 OK: 작업이 실행되었으며 응답 메시지에 상태를 설명하는 representation이 포함되어 있다.

DELETE 요청에 대한 응답은 캐시 불가능하다. 성공적인 DELETE 요청이 캐시를 통과하면, 해당 대상 URI에 대해 저장된 응답 캐시들은 모두 무효화된다. 

### PATCH

PATCH 메소드는 Request-URI로 식별되는 리소스에 set of changes를 적용하도록 요청한다. set of changes는 "patch document"라 불리는 미디어 타입에 의해 식별된다. URI가 존재하지 않는 리소스를 가리킬 경우, patch document 유형, 권한 등을 기준으로 판단하여 새 리소스를 생성할 수 있다. 

PATCH 메소드는 URI로 식별된 대상 리소스 외의, 다른 리소스들에도 side effect를 일으킬 수 있다. 

여러 PATCH 요청이 충돌할 경우 리소스를 파괴할 수 있다. 이러한 PATCH 요청을 사용할 경우, 클라이언트는 조건부 요청을 사용하여 마지막 리소스 접근 이후에 리소스가 업데이트된 경우 요청이 실패하도록 한다. 

서버는 set of changes를 원자적으로 적용해야 하며, (e.g., PATCH 처리 중 GET 요청이 들어오더라도) 부분적으로 수정된 representation을 제공해서는 안된다. 

특정 조건을 만족하는 경우 PATCH 응답은 캐시될 수 있다. PATCH 요청이 캐시를 통과하면, Request-URI에 대해 저장된 응답 캐시들은 모두 무효화된다. 



## 참고자료

[1] HTTP Semantics [RFC9110](https://httpwg.org/specs/rfc9110.html)

[2] PATCH Method for HTTP [RFC5789](https://httpwg.org/specs/rfc5789.html)

[3] Should HTTP PUT create a resource if it does not exist? [stackoverflow comment](https://stackoverflow.com/a/56241716/8315523)

[4] 당신의 PUT은 리소스를 생성하면 안된다 [velog.io/@sangmin7648/당신의-PUT은-리소스-생성하면-안된다](https://velog.io/@sangmin7648/%EB%8B%B9%EC%8B%A0%EC%9D%98-PUT%EC%9D%80-%EB%A6%AC%EC%86%8C%EC%8A%A4-%EC%83%9D%EC%84%B1%ED%95%98%EB%A9%B4-%EC%95%88%EB%90%9C%EB%8B%A4)

