# Registry 认证和授权


## 认证方案

Registry的授权方案是基于OAuth2.0的密码模式。

![v2-registry-auth](https://docs.docker.com/registry/spec/images/v2-registry-auth.png)

上面代表的是以Docker举例的Registry的认证鉴权方案：

1.Docker Daemon尝试向Registry发起请求。

2.如果Regsitry要求认证的话，会返回`401 Unauthorized`并带有认证服务的信息。

3.Docker Daemon向Authrization Service请求token。

4.Authrization Service返回客户端认真后的权限。

5.Docker Daemon将token放在header的Authorization的字段中再次向Registry发起请求。

6.Regestry解析token，并根据token中包含的权限信息开始push或者pull的会话连接。

## 附录

### OAuth2.0

#### OAuth2.0与session、cookie机制对比

- 与session机制类似，OAuth2.0只是变成token，但是session有其局限性，特别是API对接。
- 还有一些终端默认是不带cookie的，比如Android。
- OAuth2.0不是一个认证协议（是授权协议），OAuth2.0本身并不能告诉你任何用户信息。

#### 四种授权模式

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

