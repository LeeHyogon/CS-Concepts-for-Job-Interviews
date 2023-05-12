### CSRF
- Cross-Site Request Forgery, 사이트 간 요청 위조
- 웹 취약점 중 하나로, 이용자가 의도하지 않은 요청을 통한 공격을 의미.
- 쿠키가 서버의 요청에 함께 전송되는 것을 이용한다.

#### 예시

은행의 웹사이트에서 현재 로그인 중인 유저가 손을 송금할 수 있는 폼은 아래와 같다고 가정.
```html
<form method="post"
      action="/transfer">
    <input type="text"
           name="amount"/>
    <input type="text"
           name="routingNumber"/>
    <input type="text"
           name="account"/>
    <input type="submit"
           value="Transfer"/>
</form>
```
이에 따른 HTTP 요청 헤더는 아래와 같다.
``` http request
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: JSESSIONID=randomid
Content-Type: application/x-www-form-urlencoded

amount=100.00&routingNumber=1234&account=9876
```
당신은 은행에서 볼일을 다 본 후 로그아웃을 하지 않고 나쁜 웹사이트에 접속을 하게된다. </br>
그런데 그 나쁜 웹페이지에 아래와 같은 HTML 페이지가 있다.

```http request
<form method="post"
action="https://bank.example.com/transfer">
<input type="hidden"
	name="amount"
	value="100.00"/>
<input type="hidden"
	name="routingNumber"
	value="evilsRoutingNumber"/>
<input type="hidden"
	name="account"
	value="evilsAccountNumber"/>
<input type="submit"
	value="Win Money!"/>
</form>
```
<form method="post"
	action="https://bank.example.com/transfer">
<input type="hidden"
	name="amount"
	value="100.00"/>
<input type="hidden"
	name="routingNumber"
	value="evilsRoutingNumber"/>
<input type="hidden"
	name="account"
	value="evilsAccountNumber"/>
<input type="submit"
	value="Win Money!"/>
</form>
다음과 같이 생긴 Win Moeny라는 버튼을 누른다면 나쁜 웹사이트 주인의 계좌로 100달러가 입금된다.

- 이런 일이 발생하는 이유는 나쁜 웹사이트는 쿠키를 볼 수 없지만, 당신의 은행과 관련된 쿠키는 여전히 요청과 함께 전송되기 때문
- 버튼 클릭 필요 없이 이런 프로세스가 자바스크립트를 사용해 자동화 가능.


#### 대표적인 방어

1. Referrer Check
- 백엔드에서 request의 referer를 확인하여 도메인이 일치하는지 검증하는 방법.
- 대부분의 CSRF 공격을 방어 가능

2. SameSite
- 최신 브라우저가 아닌 경우는 지원하지 않을 수 있음.
- HTTP 쿠키에서 CSRF 공격의 방지를 위해 넣을 수 있는 설정. 3가지 옵션 None, Lax, Strict가 있다.
- None: 쿠키가 모든 상황에서 전송됨.
- Lax: 쿠키가 동일한 사이트(same-site)에서의 요청이 있을 때 혹은 top-level navigation을 통해 요청이 보내질 때 쿠키를 전송됨.
- strict: 쿠키가 동일한 사이트 내의 요청으로만 전송됨.

#### 3. CSRF Token

- 서버에서는 뷰 페이지를 발행할 때 랜덤으로 생성된 token을 같이 준 뒤, 사용자 세션에 저장.
- 사용자가 서버에 작업을 요청할 때, 페이지에 Hidden으로 숨어있는 token값이 같이 세션에 저장된 값과 일치하는지 확인하여 요청 위조 여부를 확인.
- 일치 여부를 확인한 token은 바로 폐기하고, 새로운 뷰 페이지를 발행할 때마다 새로 생성.

#### CSRF Token의 단점?

- Token은 세션에 정보를 저장하면서 생기는 stateful, 오버헤드 문제를 해결하기 위해 사용하는 것이다.
- 그런데, CSRF token은 세션에 저장하므로 세션의 문제점을 그대로 계승하는 것 같다는 생각이 들었다.
- 검색을 통해서는, 이러한 문제에 대한 언급을 하는 정보를 찾을 수 없었음.

#### Spring security의 interface CsrfRepository의 구현체 

1.CookieCsrfTokenRepository

```java
@Override
	public void saveToken(CsrfToken token, HttpServletRequest request, HttpServletResponse response) {
		String tokenValue = (token != null) ? token.getToken() : "";
		Cookie cookie = new Cookie(this.cookieName, tokenValue);
		cookie.setSecure((this.secure != null) ? this.secure : request.isSecure());
		cookie.setPath(StringUtils.hasLength(this.cookiePath) ? this.cookiePath : this.getRequestContext(request));
		cookie.setMaxAge((token != null) ? this.cookieMaxAge : 0);
		cookie.setHttpOnly(this.cookieHttpOnly);
		if (StringUtils.hasLength(this.cookieDomain)) {
			cookie.setDomain(this.cookieDomain);
		}
		response.addCookie(cookie);
	}
```
2. HttpSessionCsrfTokenRepository

```java
@Override
	public void saveToken(CsrfToken token, HttpServletRequest request, HttpServletResponse response) {
		if (token == null) {
			HttpSession session = request.getSession(false);
			if (session != null) {
				session.removeAttribute(this.sessionAttributeName);
			}
		}
		else {
			HttpSession session = request.getSession();
			session.setAttribute(this.sessionAttributeName, token);
		}
	}
```

- 스프링 시큐리티의 default csrf token repository는 seesion이다.
- 더이상 세션이 생성되지 않거나 세션이 없을 경우, cookieRepository를 사용하는 방식으로 진행된다.


ref https://velog.io/@dong5854/CSRF%EB%9E%80 </br>
ref https://docs.spring.io/spring-security/reference/features/exploits/csrf.html