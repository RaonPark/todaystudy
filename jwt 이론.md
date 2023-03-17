# JWT

- JWT는 RFC 7519를 따르는 표준이다.
- 서명은 HMAC 알고리즘을 사용한 SECRET이나 RSA나 ECDSA를 사용한 public/private key pair를 사용한다.
- AUTHORIZATION
    - 한 번 로그인 하면 JWT를 발급하고, 그 토큰을 가지고 서비스에 접근할 수 있도록 한다.
- INFORMATION EXCHANGE
    - JWT 토큰은 서명이 되어야하므로 상호간 정보 전달 시에 좋다.

## JWT Web Token

### Header

- 보통은 두 부분으로 이뤄진다. token의 타입과 서명 algorithm이다.

```json
{
	"alg": "HS256",
	"typ": "JWT"
}
```

- 이 JSON은 Base64Url 인코딩이 되어 JWT의 첫 부분에 들어가게 된다.

### Payload

- claims를 포함한다. claim은 entity와 additional data에 대한 statement이다.
    - Registered Claims : 미리 정의된 claim의 집합으로, 의무적이지 않지만 추천된다.
        - iss(issuer, 발행인), exp(expiration time, 마감 기한), sub(subject, 주제), aud(audience, 토큰 대상자)을 포함한다.
    - Public Claims : JWT를 사용하는 사람이 마음대로 정할 수 있는 claim이다. 하지만 충돌을 피하기 위해서 IANA JSON Web Token Registry나 URI로 정의되어야 한다.
    - Private Claims : reigstered claim이나 public claim을 사용하는 당사자들끼리 정보를 전달하기 위한 custom claim을 의미한다.

```json
{
	"sub": "1234567890",
	"name": "John Doe",
	"admin": true
}
```

- Base64Url로 인코딩되며, JWT의 두 번째 부분이다.
- payload나 header에 암호화되지 않은 비밀 정보를 넣는 것은 누구나 볼 수 있기 때문에 좋지 않다.

### Signature

- 암호화된 header, payload, secret등을 위한 서명 부분이다.
- 만약, HMAC SHA256알고리즘으로 암호화하면, signature은 다음과 같다.

```java
HMACSHA256(
	base64UrlEncode(header) + "." +
	base64UrlEncode(payload),
	secret)
```

- signature는 메세지를 확인하기 위해 사용되며, 토큰이 private key로 서명된 경우, JWT를 보낸 사람을 확인하는데도 사용할 수 있다.

![](https://cdn.auth0.com/content/jwt/encoded-jwt3.png)

- 실제로 header, payload를 인코딩하여 secret으로 서명한 JWT이다.

## JWT 동작 방법

- 유저가 credential을 가지고 로그인 하는 경우, JWT가 반환된다. 토큰이 credential이므로, 토큰을 잘 관리해야한다. 보통은 이 토큰을 필요 이상으로 유지하는 것은 좋지 않다.
- 보안된 루트나 리소스를 접근하는 경우, JWT를 보내줘야하며, 보통은 Authorization header에 Bearer 스키마를 사용한다.

```json
Authorization: Bearer <token>
```

- JWT 토큰을 HTTP 헤더로 보내는 경우, 너무 커지지 않도록 예방하는 것이 좋다. 몇몇 서버는 8KB가 넘지 않도록 하는 것도 있다.
- 토큰이 Authorization 헤더로 보내지면, CORS(Cross-Origin Resource Sharing)은 쿠키를 사용하기 때문에 일어나지 않는다.

![](https://cdn2.auth0.com/docs/media/articles/api-auth/client-credentials-grant.png)

1. 애플리케이션이나 client가 authorization을 authorization server에 요청한다. 
2. authorization이 허용되면 authorization server는 access 토큰을 반환한다.
3. application은 access token을 보안 리소스에 접근하는데 사용한다.
