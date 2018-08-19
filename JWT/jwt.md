# JWT

## 一| JWT的组成

一个JWT实际上就是一个字符串，它由三部分组成，头部、载荷与签名。

格式：

```json
header.payload.sign
```

## 二| 头部（Header）

```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

## 三| 载荷（Payload）

由一系列 Claim Names 组成：

### 3.1 Registered Claim Names

- "iss" (Issuer) Claim: 可选，签发者
- "sub" (Subject) Claim: 可选，主题
- "aud" (Audience) Claim: 可选，接收者
- "exp" (Expiration Time) Claim: 可选，到期时间
- "nbf" (Not Before) Claim: 可选，在某时间之前不生效（该时间之后才生效）
- "iat" (Issued At) Claim: 可选，签发时间
- "jti" (JWT ID) Claim: 可选，jwt唯一标识

### 3.2 Public Claim Names

JWT 使用者可随意定义的。

### 3.3 Private Claim Names

生产者和消费者共同约定的，非 Registered 非 Public 的 Claim Names。

## 四| 签名（Signature）

`头部.载荷`

将上面拼接完的字符串用HS256算法进行加密。在加密的时候，我们还需要提供一个密钥（secret）。

## 五| 参考

- [JSON Web Token (JWT)](https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32)
- [JSON Web Token - 在Web应用间安全地传递信息](https://blog.leapoahead.com/2015/09/06/understanding-jwt/)