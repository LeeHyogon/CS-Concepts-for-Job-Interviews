### Token 인증

- 클라이언트가 서버에 접속 하면 서버에서 해당 클라이언트에게 인증되었다는 의미로 '토큰'을 부여한다.
- 이 토큰은 **유일**하며 클라이언트는 다시 서버에 요청 헤더에 토큰을 심어서 보낸다. 그러면 서버에서 제공한 토큰과 일치 여부를 체크하여 인증 과정을 처리한다.

#### 장점
- 서버가 아닌 클라이언트에 저장되기 때문에 서버의 부담이 줄음.
- 상태를 유지하지 않으므로 Stateless.
- 웹에는 쿠키와 세션이 있지만, 앱에는 없다.

#### 단점

- 토큰 자체의 데이터 길이가 길어, 인증 요청이 많아지면 네트워크 부하가 심해진다.
- Payload자체는 암호화되지 않기 때문에, 유저의 중요한 정보는 담을 수 없다.
- 토큰을 **탈취**당하면 대처하기 어렵다.

### JWT

- 인증에 필요한 정보들을 암호화시킨 JSON 토큰을 의미한다.
- JSON데이터를 인코딩 하여 직렬화 한 것이며, 변조 방지를 위해 개인키를 통한 전자서명도 들어있다.


#### JWT 구조

- Header: JWT에서 사용할 타입과 해시 알고리즘의 종류가 담겨있다.
- Payload: 서버에서 첨부한 사용자 권한 정보와 데이터가 담겨있다.
- Signature: Header, Payload를 BASE64URL로 encode한 이후 Header에 명시된 해시함수를 적용하고, 개인키로 서명한 전자서명이 담겨있다.

![](https://velog.velcdn.com/images/gon109/post/22b3bd3f-cd08-4fcc-bb87-f9cf66c67bbb/image.png)


#### 장점

- 데이터 위변조를 막을 수 있다.
- 인증 정보에 대한 별도의 저장소가 필요없다.
- 서버가 Stateless가 되어 서버 확장성이 우수해진다.
- 토큰 기반으로 **다른 로그인 시스템에 접근 및 권한 공유**가 가능하다.
- 모바일 환경에서도 사용 가능하다.

#### 단점
- 토큰 길이: 네트워크에 부하를 줄 수 있다.
- Payload 인코딩: payload 자체는 암호화 된 것이 아니므로, payload에 중요 데이터를 넣지 않아야 한다.
- Store Token: 클라이언트에서 토큰을 관리하므로, 토큰 자체가 탈취 당하면 대처가 어렵다.


### Access Token & Refresh Token

- 토큰 탈취의 위험성 때문에, 그대로 사용하지 않고 Acees Token,Refresh Token으로 이중으로 나누어 인증을 하는 방식이 권장된다.

#### Access Token
- 클라이언트가 갖고있는 유저 정보가 담긴 토큰. 토큰의 유효 시간을 짧게 부여하여, 탈취 문제에 대응한다.

#### Refresh Token
- 새로운 Access Token을 발급해주기 위해 사용하는 토큰
- 짧은 수명을 가지는 Access Token을 재발급하여 로그인을 자주 하지 않게 한다.


#### case)
- case1 : access token과 refresh token 모두가 만료된 경우 → 에러 발생 (재 로그인하여 둘다 새로 발급)
- case2 : access token은 만료됐지만, refresh token은 유효한 경우 →  refresh token을 검증하여 access token 재발급
- case3 : access token은 유효하지만, refresh token은 만료된 경우 →  access token을 검증하여 refresh token 재발급
- case4 : access token과 refresh token 모두가 유효한 경우 → 정상 처리


#### Refresh Token 탈취 문제
- Access Token을 짧게 발행하여, 탈취 문제를 해결한다 하더라도 refresh token의 탈취 문제가 있다.
- 즉 XSS, CSRF공격으로부터 안전하게 보관해야 한다.

### Refresh Token의 저장 위치

#### Front
- Local Storage: XSS 공격에 취약.
- Cookie: 기본적으론 XSS,CSRF 공격에 취약하나 쿠키에 httpOnly / secure / SameSite 옵션을 사용한다면 자바스크립트 코드 상에서 접근이 불가능해지고, HTTP 요청에만 포함되어 보내진다.

#### Back

- Session: 기존 세션 방식보다 토큰의 사이즈가 훨씬 크므로 서버 부하 문제가 더 심해진다. 또한 웹 서버에서 관리하므로, 공격의 위험성이 있다.
- DB: refresh token을 DB에 저장한 후, index 값을 쿠키나 로컬스토리지에 저장하는 방법. access token이 짧은 주기로 만료되는데, 매번 DB를 접근하는 건 비효율적이다.

#### Redis
- redis는 in-memory DB임에도 불구하고, 메모리 데이터를 disk에 저장할 수 있는 특징으로, 영속성 기능이 지원된다.
- DB 접근의 비효율적인 문제를 캐시로 해결하고, 만료 기간도 설정가능하므로 refresh token을 저장하기 좋다.




ref https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-JWTjson-web-token-%EB%9E%80-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC <br/>
ref https://hudi.blog/refresh-token/