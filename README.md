## JWT

**Authorization Middleware for Caddy**

This middleware implements an authorization layer for [Caddy](https://caddyserver.com) based on JSON Web Tokens (JWT).  You can learn more about using JWT in your application at [jwt.io](https://jwt.io).

### Syntax


```
jwt [path]
```

By default every resource under path will be secured using JWT validation.  To specify a list of resources that need to be secured, use:

```
jwt {
path [path1]
path [path2]
}
```

> **Important** You must set the secret used to construct your token in an environment variable named `JWT_SECRET`.  Otherwise, your tokens will always silently fail validation.  Caddy will start without this value set, but it must be present at the time of the request for the signature to be validated. 

### Ways of passing a token for validation

There are three ways to pass the token for validation: (1) in the `Authorization` header, (2) as a cookie, and (3) as a URL query parameter.  The middleware will look in those places in the order listed and return `401` if it can't find any token.

##### Authorization Header
```
Authorization: Bearer <token>
```

##### Cookie
```
"jwt_token": <token>
```

##### URL Query Paramter
```
/protected?token=<token>
```

### Constructing a valid token

JWTs consist of three parts: header, claims, and signature.  To properly construct a JWT, it's recommended that you use a JWT library appropriate for your language.  At a minimum, this authorization middleware expects the following fields to be present:

##### Header
```json
{
"typ": "JWT",
"alg": "<any supported algorithm except none>"
}
```

##### Claims
```json
{
"exp": "<expiration date as a Unix timestamp>"
}
```

### Acting on claims in the token

You can of course add extra claims in the claim section.  Once the token is validated, the claims you include will be passed as headers to a downstream resource.  Since the token has been validated by Caddy, you can be assured that these headers represent valid claims from your token.  For example, if you include the following claims in your token:

```json
{
"user": "test",
"role": "admin",
"logins": 10
}
```

The following headers will be added to the request that is proxied to your application:

```
Token-Claim-User: test
Token-Claim-Role: admin
Token-Claim-Logins: 10
Token: <full token string>
```

Token claims will always be converted to a string.  If you expect your claim to be another type, remember to convert it back before you use it.  The full token string is always passed as `Token`.

### Caveats

JWT validation depends only on validating the correct signature and that the token is unexpired.  You can also set the `nbf` field to prevent validation before a certain timestamp.  Other fields in the specification, such as `aud`, `iss`, `sub`, `iat`, and `jti` will not affect the validation step.
