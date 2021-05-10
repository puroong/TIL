# CORS Request with Cookie

한 컨설팅 업체에서 LMS 서비스를 개발하고 있던 도중 액세스 토큰을 쿠키로 전달하는 방식을 채택하게 됐음

매번 액세스 토큰을 Authorizatoin 헤더로만 보내다 쿠키로 보내면서 배운 점을 정리해보려 함

## CORS(Cross-Origin-Resource-Sharing)란

* 기본적으로 브라우저는 보안 상의 이유로 다른 origin(도메인, 스키마, 포트)으로 요청을 보낼 수 없음
    * 이를 허용하려면 서버에 적절한 헤더를 설정해줘야 함

* 실제 요청을 보내기 전에 실제 요청에 사용될 헤더와 HTTP 메소드를 헤더에 넣고 요청을 보내서 서버가 CORS 요청을 허용하는지 확인함 (preflight request)

* 사이드 이펙트를 발생시키는 메소드들(GET, POST 외 나머지 메소드들)은 반드시 preflight request를 먼저 보내야함

### CORS 요청 예시

#### Simple requests

* cors 요청이 아래 조건에 모두 부합하는 경우, preflight request를 보내지 않아도 됨
    1. `GET`, `HEAD`, `POST` 메소드 중 하나
    2. Uesr Agent가 설정한 헤더를 제외하고 개발자가 직접 설정한 헤더들이 `Accept`, `Accept-Language`, `Content-Launguage`, `Content-Type` 중 하나여야 함
    3. `Content-Type`에 올 수 있는 값은 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 중 하나임
    4. `XMLHttpRequest`로 요청을 보내는 경우 `XMLHttpRequest.upload`의 이벤트 리스너가 설정되면 안됨
    5. 요청에 `ReadableStream` 객체가 사용되면 안됨

* preflight request를 보내지 않더라도 응답에 적절한 헤더가 없으면 에러 발생함
    * 아래 스크린샷을 통해 prefligth request를 보내지 않은 것을 확인할 수 있음

실패

![](/assets/simple-request-cors-error.png)

성공

![](/assets/simple-request-success.png)

* 응답의 `Access-Control-Allow-Origin`와 요청의 `Origin` 헤더 값을 비교함

* credential이 포함된 요청일 경우 `Access-Control-Allow-Origin`에 origin을 명시해줘야함 **(`*` 못 씀)**

#### Preflighted requests

* Simple requests와 달리 이 요청은 실제 요청을 보내기 전에 동일한 url에 OPTION 메소드로 요청을 보내고, 응답으로 받은 헤더 값을 통해 실제 요청을 보내도 되는지 판단함

* preflight request 보낼 때 `Access-Control-Request-Method`에 실제 요청 메소드를 보내고, `Origin`에 origin을, 그리고 커스텀 헤더를 사용하는 경우 `Access-Control-Request-Headers`에 헤더 목록을 보냄

* 응답에 `Access-Control-Max-Age` 헤더를 설정하면 설정된 시간동안 preflight request가 캐싱되며 실제 요청을 보내기 전에 preflight request를 보내지 않아도 됨
    (단위는 `초`임)

Preflighted requests and redirects

* 몇몇 브라우저들이 preflighted request 이후에 리다이렉트 하는 기능을 지원하긴 하지만 모든 브라우저들이 지원을 하진 않습니다
    (원래 cors 프로토콜은 리다이렉트를 못하게 했는데, 중간에 허용하는걸로 변경됐음. 하지만 모든 브라우저들이 이 변화를 반영하지 않았음)

* 만약 prefligeted request 이후에 리다리렉트 해야 한다면 아래 방법들 중 하나를 사용할 수 있음
    1. 서버 쪽에서 preflight request를 안보내거나 리다이렉트를 안하도록 설정함
    2. simple request를 사용하도록 수정함


#### Requests with credentials

* XMLHttpRequest 또는 Fetch 로 cors 요청을 보낼 때 credential(HTTP 쿠키나 HTTP 인증 정보)를 같이 보낼 수 있음

* 기본적으로 cors 요청 보낼 땐 credential을 포함하지 않는데 withCredential 옵션을 설정하면 같이 보낼 수 있음

* credential을 같이 보내기 위해선 응답에 `Access-Control-Allow-Credentials` 헤더 값이 true여야 함

#### Preflight request and crednetials

* preflight request는 credential을 포함하면 안되며 응답엔 반드시 `Access-Control-Allow-Credentials` 헤더가 있어야함

#### Credential requests and wildcards

* credential을 포함한 요청을 보낸 경우 `Access-Control-Allow-Origin`에 `*`를 사용할 수 없음

* credential을 포함한 요청을 보낼 경우 `Access-Control-Allow-Methods`에 `*`를 사용할 수 없음

## 쿠키

### 쿠키 종류

* First-party Cookie
* Third-party Cookie

## First-party Cookie

* 직접 방문한 웹사이트가 설정한 쿠키

## Third-party Cookie

* 직접 방문하지 않은 웹사이트가 설정한 쿠키

## Third-party Cookie 생성 방법

* 유저가 접속한 웹사이트에서 다른 서버로 요청을 보내야 함