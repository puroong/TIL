# Access-Control-Allow-Methods와 Credentail

쿠키로 액세스 토큰을 api로 전달하면 인증을 수행하고 있었는데, Access-Control-Allow-Methods 값을 *로 설정했음에도 불구하고 

PUT과 DELETE 요청 전송 시 CORS 에러가 발생했습니다

*는 credential을 포함하지 않은 요청에 대해서만 와일드카드 역할을 수행합니다

만약 이번처럼 쿠키를 보낼 땐 와일드카드가 아닌 `*` 메소드로 취급됩니다