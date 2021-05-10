# Set-Cookie 속성

## Set-Cookie

* User Agent에게 쿠키를 전달할 때 사용하는 헤더

* 브라우저에서 보안 문제때문에 js로 `Set-Cookie`헤더 값을 접근할 수 없음

## 속성 목록

### <cookie-name>=<cookie-value>

* 쿠키 이름이 `_Secure-`로 시작한다면 secure 플래그를 설정해줘야 함

* 쿠키 이름이 `_Host-`로 시작한다면 secure 플래그를 설정해줘야 하고, 도메인 속성이 설정되지 않아야 하며, path는 `/`이어야 함

### Expires=<date>

* 쿠키가 만료되는 날짜

* 만약 명시되지 않으면 세션 쿠키로 취급됨

    (브라우저가 닫히면 사라짐)

### Max-Age=<number>

* 쿠키 유효기간을 초 단위로 표현한 값

* `Expires`와 동시에 설정되면 `Max-age`값을 사용함

### Domain=<domain value>

* 기본값은 현재 도메인

* 멀티 호스트 지원안함 (단, 주어진 도메인의 서브도메인들은 항상 포함됨)

### Path=<path-value>

### Secure

* 쿠키를 `https` 프로토콜로만 전송할 수 있음

    (`https`로 전송하더라도 insecure한 사이트들도 쿠키를 설정할 수 없음)

### HttpOnly

* js에서 쿠키로 접근하지 못하도록 막음

* xss 공격을 방지할 수 있음

### SameSite=<samesite-value>

* cors 요청 보낼 때 쿠키를 같이 보낼지 결정함

* csrf 공격을 방지할 수 있음

#### SameSite-Value 목록

* Strict
    * same-site 요청 보낼 때만 쿠리를 같이 보냄
* Lax
    * cross-site 요청엔 쿠키를 포함하지 않지만 (이미지, 프레임 로딩할 때), origin 사이트에서 링크를 타고 외부 사이트로 이동할 땐 쿠키를 같이 보냄
        
        (기본값임)
* None
    * smae-site & cross-site 요청 모두 쿠키를 같이 보냄