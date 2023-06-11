# Registry 认证和授权


## 认证方案

Registry的授权方案是基于OAuth2.0的密码模式。

![v2-registry-auth](https://docs.docker.com/registry/spec/images/v2-registry-auth.png)

上面代表的是以Docker举例的Registry的认证鉴权方案：

1.Docker Daemon尝试向Registry发起请求。

2.如果Regsitry要求认证的话，会返回`401 Unauthorized`并带有认证服务的信息。

3.Docker Daemon向Authrization Service请求token。

4.Authrization Service返回客户端认证后的权限。

5.Docker Daemon将token放在header的Authorization的字段中再次向Registry发起请求。

6.Regestry解析token，并根据token中包含的权限信息开始push或者pull的会话连接。

## 请求token

本小节的内容是关于上图中的步骤3的详解。

在请求Token的API中，有以下参数：

### Query Parameters

- service：（neccessary）授权服务的标识，表示要向谁请求token
- scope：（neccessary）
- client-id：（optinal）请求token的客户端id，比如docker-daemon发起的请求会将该字段设置为docker。

### Header Parameters

- uthorization：（optional）携带的用户信息

### Response Body

响应body为一个json，有三个字段:

- token：（neccessary）授权服务器返回的带有授权信息的token。
- issued_at：（optional）token的签发时间，UTC标准时间格式。
- expires_in：（optional）token在多少秒以后过期，如果没有说明则默认为60秒。

### 示例

```sh
$ curl 192.168.1.103:8021/service/token?service=token-service\&scope=repository:library/registry:pull\&client_id=curl
{
  "expires_in": 1800,
  "issued_at": "2018-09-05T08:34:40Z",
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IkhNNjY6NkNYUzpaQlBROk1ENVo6QlJZVTpTVE9EOkNCUEs6Uk5ORjpYN0VDOkZMUUw6TFNFMjpLUUtTIn0.eyJpc3MiOiJyZWdpc3RyeS10b2tlbi1pc3N1ZXIiLCJzdWIiOiIiLCJhdWQiOiJ0b2tlbi1zZXJ2aWNlIiwiZXhwIjoxNTM2MTM4MjgwLCJuYmYiOjE1MzYxMzY0ODAsImlhdCI6MTUzNjEzNjQ4MCwianRpIjoiZFpxVkgxVDFjZkhXdnFZTiIsImFjY2VzcyI6W3sidHlwZSI6InJlcG9zaXRvcnkiLCJuYW1lIjoibGlicmFyeS9yZWdpc3RyeSIsImFjdGlvbnMiOlsicHVsbCJdfV19.ilCKa2-oJ9bKQpAo8ntcx1lHpbs0BcWYtbRrvItHAProaDEDpll9EZrzkzg6XR9OOLByFm_oJKKk8Y_wYwQfxYdjvhLbFjCNXzE6MckY8dEcSR5BmYxOK54zAqNVkw24ugUcagGFi7p8Gy0YZqBqf7AP8qCarhuWhKsZ7B4esMQk2xBEn1hh8r_9tb6wnZOkDl7trW0IWbPkqKSaP8ycq8oS9J0T6zaItyTLnERsV_GFJOh6DdfhSYzGwoWUFQH6cmp05ZHXF_-4O6N6d8tosGH9gTsam-ffeVHmWp8da_gpS_R15z3ELR5I2FO0s4gWo1UbTI3yuyV8stSURrCs6GHZSMb2C9_2R2r_Q-uDKmdpoazw2G1DxM3PgfXEwANWEjPJMjD0areUXmjwz_hefSMqYFxLi26TaQinG0th7pNz5m0qroefOy1AGyhRZK-t8rsduZJ9EWQCqtXHrPbTES0FoItJmcMqcJZcvQsrJsBMirtijvGdNn55l44-eDFyrIuExerHzU1dJoSijCtqIYxbdnclLE8HSP-vnBD5TOAJoUUdUfA1N8TvF2QqDjr_LATUOctahrFoWiuDjrFXH-ptmcJJ6lPjo1oCOne3ImKe_mieRR7YCOQLejuCbItIIweuqwBzJU5d33k3Drra0qvbvk-MkO7iBNgpCtfWqD8"
}
```

我们可以把上面的token拷贝到网页jwt.io中，查看token的明文形式。
在上面获取token的请求中，没有携带任何的用户信息。不过我们可以使用添加用户信息（用户名与密码）去获取token，如下：

```sh
$ curl -H "Authorization: Basic YWRtaW46SGFyYm9yMTIzNDU=" 192.168.1.103:8021/service/token?service=token-service\&scope=repository:library/registry:pull\&client_id=curl

其中YWRtaW46SGFyYm9yMTIzNDU=是admin:Harbor12345的base64编码
```

## 生成token

本小节主要是分析Authrization Service生成JWT的过程。

token由三部分内容组成：Header、Payload和Signature。token的形式如下：

```sh
{token-header}.{token-payload}.{token-signature}
```

### Header

Header有三个字段：

- typ：固定为JWT
- alg：签名算法，常用的有HS256、RS256等
- kid：key-id，签名算法中所使用的密钥的ID值

kid的生成有以下三个步骤:

1、从签名算法使用的密钥中得到DER编码格式的公钥（public key

2、对DER格式的公钥做sha256哈希，取前240bit

3、将这240bit使用base32编码，然后四个一组使用冒号:分隔

如下是Header的一个例子:

```json
{
    "typ": "JWT",
    "alg": "RS256",
    "kid"："HM66:6CXS:ZBPQ:MD5Z:BRYU:STOD:CBPK:RNNF:X7EC:FLQL:LSE2:KQKS"
}
```

生成kid的详细例子见本文末尾的扩展阅读。

### Payload

payload中的字段有：

- iss：（Issuer），token的签发者
- sub：（Subject），正在进行认证的用户的名字，如果是匿名用户则为空
- aud：（Audience），token的观众，即需要对token进行验证的服务的名字
- exp：（Expiration），过期时间，在这之后token应该看作是无效的；时间戳格式
- nbf：（Not Before），token有效的超始时间，在这之前token应当看作是无效的；时间戳格式
- iat：（Issued At），签发时间；时间戳格式
- jti：（JWT ID），token的id，（尚不清楚如何生成）
- access：权限集，下面还有三个字段
  - type
  - name
  - actions

payload的样例如下：

```json
{
 "iss": "registry-token-issuer",
 "sub": "",
 "aud": "token-service",
 "exp": 1536204479,
 "nbf": 1536202679,
 "iat": 1536202679,
 "jti": "KF76FTQQ4tIvvCbR",
 "access": [
  {
   "type": "repository",
   "name": "library/registry",
   "actions": [
    "pull"
   ]
  }
 ]
}
```

### Signature

#### Header处理

对header内容去掉空白字符后得到：

```sh
{
  "typ":"JWT",
  "alg":"RS256",
  "kid":"HM66:6CXS:ZBPQ:MD5Z:BRYU:STOD:CBPK:RNNF:X7EC:FLQL:LSE2:KQKS"
}
```

然后对该字符串进行base64Url编码（base64在线编码网址），得到token-header，base64Url就是先进行base64编码，再把得到的字符串中的+变成-，/变成_，去掉=。

```sh
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IkhNNjY6NkNYUzpaQlBROk1ENVo6QlJZVTpTVE9EOkNCUEs6Uk5ORjpYN0VDOkZMUUw6TFNFMjpLUUtTIn0
```

#### Payload处理

payload内容去掉空白字符后得到：

```sh
{
  "iss":"registry-token-issuer",
  "sub":"","aud":"token-service",
  "exp":1536204479,
  "nbf":1536202679,
  "iat":1536202679,
  "jti":"KF76FTQQ4tIvvCbR",
  "access":
    [
      {
        "type":"repository",
        name":"library/registry",
        "actions":["pull"],
      }
    ]
}
```

然后对该字符串进行base64Url编码，得到token-payload：

```sh
eyJpc3MiOiJyZWdpc3RyeS10b2tlbi1pc3N1ZXIiLCJzdWIiOiIiLCJhdWQiOiJ0b2tlbi1zZXJ2aWNlIiwiZXhwIjoxNTM2MjA0NDc5LCJuYmYiOjE1MzYyMDI2NzksImlhdCI6MTUzNjIwMjY3OSwianRpIjoiS0Y3NkZUUVE0dEl2dkNiUiIsImFjY2VzcyI6W3sidHlwZSI6InJlcG9zaXRvcnkiLCJuYW1lIjoibGlicmFyeS9yZWdpc3RyeSIsImFjdGlvbnMiOlsicHVsbCJdfV19
```

#### signature处理

token-signature的计算方法如下，先对token-header + "." + token-payload做sha256哈希（RS256就是RSA+SHA256），然后再使用RSA的私钥进行签名（sign），最后用base64Url进行编码，得到signature-token：

```sh
token-signature = base64Url(sign(sha256(token-header+"."+token-payload)))
```

由前面的token-header与token-payload得到的token-signature如下（RSA密钥见扩展阅读）:

```sh
e91bTpXSYNUTcUXr7zs62ZgCm1L6bhZbbW4ujXFY9Zzdkvy3DEHDssq6R4K9f5ESvv_LrWxxIxIXVREAATw-FaykcAewyjarC6Vlj2g0ea6D9L1HsIvsqtYcBOnHIJ5CRPJPhXWPwBtbujgNgbti-LLeVprOwaJ8fDk21UikmYFhX61_IobFukWw1ByXiNt8byU6tOrxkkDp-YXpz9y-XP5FdheGwNxOREph40znA9LddUcEuQUHB5WKQ3tdU4sqXOW3TUCtjLOl-kVREcus-83fLSuob1lZWRbzU9dEROd_5ZP4NNmD4ZY0DhcYbp75UqvB-MZIiC9MDeOheHAsPGB4Kqu2gBshRd_NJIrQkig7yvD2Wo7twn1KKSznHp6lcsK5phkkkWMVbZoD3qV76MqCDKVSkD2JOgQ0l4AhcYEGLtxx_ukk4NlDCYoljnGPw1oEynmFDROSvMg_bqhRVUF-5US83sU0l6YWwRCZT6StTvdSHp79wbSXgEn58-NO64AtVuMEb1XiDhDxtgaF0K61UwjBRmhpcCurw0laknBVVlta6otbfQcbyQn6ulsKgbrBKka-vkgo4_ymCyqnSXuZYC2Oz_PYawgGVz3s4JXhedoVWiUSDbyKYnFTXdtTign5oT6H6N-K1YKLoGSxma3uUwdDZP2hKH_UH_V9eOY
```

最后，对token-header、token-payload和token-signature进行组装，得到最终的token:

```sh
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IkhNNjY6NkNYUzpaQlBROk1ENVo6QlJZVTpTVE9EOkNCUEs6Uk5ORjpYN0VDOkZMUUw6TFNFMjpLUUtTIn0.eyJpc3MiOiJyZWdpc3RyeS10b2tlbi1pc3N1ZXIiLCJzdWIiOiIiLCJhdWQiOiJ0b2tlbi1zZXJ2aWNlIiwiZXhwIjoxNTM2NTY4NDcxLCJuYmYiOjE1MzY1NjY2NzEsImlhdCI6MTUzNjU2NjY3MSwianRpIjoiaU1kYm5td2dLQ2dUTk4xdyIsImFjY2VzcyI6W3sidHlwZSI6InJlcG9zaXRvcnkiLCJuYW1lIjoibGlicmFyeS9yZWdpc3RyeSIsImFjdGlvbnMiOlsicHVsbCJdfV19.e91bTpXSYNUTcUXr7zs62ZgCm1L6bhZbbW4ujXFY9Zzdkvy3DEHDssq6R4K9f5ESvv_LrWxxIxIXVREAATw-FaykcAewyjarC6Vlj2g0ea6D9L1HsIvsqtYcBOnHIJ5CRPJPhXWPwBtbujgNgbti-LLeVprOwaJ8fDk21UikmYFhX61_IobFukWw1ByXiNt8byU6tOrxkkDp-YXpz9y-XP5FdheGwNxOREph40znA9LddUcEuQUHB5WKQ3tdU4sqXOW3TUCtjLOl-kVREcus-83fLSuob1lZWRbzU9dEROd_5ZP4NNmD4ZY0DhcYbp75UqvB-MZIiC9MDeOheHAsPGB4Kqu2gBshRd_NJIrQkig7yvD2Wo7twn1KKSznHp6lcsK5phkkkWMVbZoD3qV76MqCDKVSkD2JOgQ0l4AhcYEGLtxx_ukk4NlDCYoljnGPw1oEynmFDROSvMg_bqhRVUF-5US83sU0l6YWwRCZT6StTvdSHp79wbSXgEn58-NO64AtVuMEb1XiDhDxtgaF0K61UwjBRmhpcCurw0laknBVVlta6otbfQcbyQn6ulsKgbrBKka-vkgo4_ymCyqnSXuZYC2Oz_PYawgGVz3s4JXhedoVWiUSDbyKYnFTXdtTign5oT6H6N-K1YKLoGSxma3uUwdDZP2hKH_UH_V9eOY
```

## 使用Token

在得到token后，我们就可以在API请求的Header中添加token信息，比如下载镜像的manifest：

```sh
curl -H "Authorization: Bearer [token]" 192.168.1.103:8021/v2/library/registry/manifests/2.5.0
```

## 验证Token

当Registry接收到一个携带token的API请求时，Registry需要从以下几个方面来验证Token:

- token的签发者（payload中的iss）是可信的，即和registry的配置参数issuer一致
- 确保registry是该token的观众，即payload中的aud与registry的配置参数token-service一致
- 检查payload中的nbf与exp确保token在有效期内
- 检查payload的access字段，确保该token能够访问该API
- 检查token的签名

## 附录

### OAuth2.0

#### OAuth2.0与session、cookie机制对比

- 与session机制类似，OAuth2.0只是变成token，但是session有其局限性，特别是API对接。
- 还有一些终端默认是不带cookie的，比如Android。
- OAuth2.0不是一个认证协议（是授权协议），OAuth2.0本身并不能告诉你任何用户信息。

##### 四种授权模式

- 授权模式（authorization code）
  
  正宗模式。认证时，直接将用户导向认证服务器，用户选择同意后，认证服务器向客户端发送授权码，客户端凭次授权码请求访问令牌，
  申请到令牌之后授权码失效，类似通过第三方软件授权。支持refresh token。

- 简化模式（implicit）
  
  比授权码模式少了授权码环节，回调url直接携带token。为Web浏览器应用设计。不支持refresh token。

- 密码模式（resource owner password credentials）
  
  用户直接把帐号密码给客户端，客户端凭此帐号密码向认证服务器请求令牌。支持refresh token。

- 客户端模式（client credentials）

  用户直接将客户端注册，客户端凭自己的名义要求认证服务提供服务。为后台API服务消费者设计。不支持refresh token。

**Refresh Token**机制用于获取新的Access Token，这样可以缩短Access Token的过期时间保证安全，同时又不会因为频繁过期重新要求用户登录。

### Token和JWT

- Token：
  
  服务端验证客户端发送过来的Token时，还需要查询数据库获取用户信息。然后验证Token是否有效。

- JWT：
  
  将Token和Payload加密后存储在客户端，服务端只需要使用密钥解密进行校验（校验也是JWT自己实现的）即可，不需要查询或者减少数据的查询，因为JWT自包含了用户信息和加密的数据。

### kid的生成

首先从RSA私钥中提取公钥，保存到文件public_key.pem中：

```sh
openssl rsa -in private_key.pem -out public_key.pem -pubout
```

然后将公钥文件由pem格式生成der格式：

```sh
openssl rsa -pubin -inform PEM -in public_key.pem -outform DER -out public_key.der
```

然后对der格式的公钥文件做sha256哈希：

```sh
sha256sum public_key.der
3b3def0af2c85f060fb90c71494dc3105ea8b5a5bfc822ae0b5c89a54152e706
```

去掉十六进制的哈希值后四位得到：

```sh
3b3def0af2c85f060fb90c71494dc3105ea8b5a5bfc822ae0b5c89a54152
```

然后用base32（RFC4648）进行编码（base32在线编码网址），编码后得到：

```sh
HM666CXSZBPQMD5ZBRYUSTODCBPKRNNFX7ECFLQLLSE2KQKS
```

每四位一组，中间用:隔开，得到kid的值

```sh
HM66:6CXS:ZBPQ:MD5Z:BRYU:STOD:CBPK:RNNF:X7EC:FLQL:LSE2:KQKS
```

