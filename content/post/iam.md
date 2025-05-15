+++
title = '一文搞懂IAM'
date = 2024-10-10T19:47:25+08:00
draft = false
+++

## 一、前言

上一份工作曾经部分iam的功能开发，但是没有深入的理解和认识IAM。碰巧最近新工作，需要有iam相关的功能。最近做了些整理，记录下。

## 二、什么是IAM

直接问GPT，下面是GTP给出的结果。

IAM（Identity and Access Management，身份与访问管理）是一套用于管理用户身份及其对资源访问权限的框架。它帮助组织控制谁（用户或设备）可以访问什么资源，并规定了在什么条件下可以访问这些资源。IAM主要有以下几个核心功能：
1. 身份管理：创建、修改和删除用户账户，确保每个用户有唯一的身份。
2. 访问控制：管理用户对系统资源的访问权限，决定用户能访问哪些资源以及可以执行哪些操作。
3. 认证：通过验证用户的身份，确保只有合法用户可以访问系统（常见认证方式有密码、双因素认证等）。
4. 授权：在用户身份得到验证后，根据分配的权限决定用户可以访问哪些资源及进行什么操作。
5. 审计和监控：跟踪和记录用户的活动，确保符合安全政策，并提供详细的日志用于事后审计。
IAM 通常用于云平台（如 AWS、Azure、GCP）以及企业内部的系统，帮助企业实现精细化的权限管理，提高系统的安全性。

IAM也常被称为”3A“或者”4A“。4A主要有以下几个模块：
- 账号（Account）
- 认证（Authentication）
- 权限（Authorization）
- 审计（Audit）
3A就是不包含审计，审计这块由单独的日志模块承载。
审计这块本文不做探讨，仅探讨3A。

### 2.1 身份管理

什么管理很简单，就是用户，更简单的理解就是账号。每一个应用基本都有个账号系统，都需要登录。不管是云服务应用，还是独立设备应用，都得有账号。可能云服务应用支持注册账号，但设备软件应用，不支持注册，都是由默认的预制超级账号admin，创建出来。

#### 2.1.1 账号管理的基本功能

1. 用户注册与创建
用户是整个系统的基本单元，一个账号对应一个用户。通常都会支持用户的增删改查。
2. 密码管理
用户可以创建修改密码、支持设置密码复杂度，支持多重密码认证等。
3. 账户生命周期管理
支持账户激活、停用、恢复等策略
4. 角色与用户组
支持归类用户，将不同类型的用户进行划分。这块有的概念叫用户组、有的概念叫工作组。
从划分的方向上来讲，可以分为横向划分和纵向划分。横向的概念是对多个同级账号进行划分，纵向的概念是将子账号进行组织。
角色的概念，实际上跟授权相关。角色是对用户所具有功能的一种权限集合抽象。常见的角色包含管理员、审计员、操作员等。
5. 属性和凭据管理
属性是指账号的个人信息，比如手机号、邮箱、图片等。这块又分为隐私数据和敏感数据。
凭据管理，通常是指账号的API密钥、证书等，这些凭据可以用于远端登录或API验证。
6. 身份认证
用户密码用于确认用户的身份，这块跟认证相关，常见的认证方式包括用户名和密码、双因子认证、生物识别认证（人脸）

#### 2.1.2 访问控制的基本功能
##### 权限管理
- **角色和权限分配**：根据用户的身份匹配不同资源的访问权限。
- **最小权限原则**：用户或系统只获得其执行其工作的最小权限，避免不必要的权限滥用
- **基于角色的访问控制（RBAC）**：通过将用户分配到不同的角色中，角色拥有预定义的权限集合，简化权限管理。
- **基于属性的访问控制（ABAC）**：根据用户的属性或资源的属性动态授予权限。

##### 会话管理
- **会话时间控制**：限制用户会话的持续时间，强制定期重新认证。
- **会话状态管理**：管理用户的登录会话，允许管理员强制终止异常或高风险的会话。

#### 2.1.3 认证方式
- 密码认证
- 多因素认证：在输入密码的情况下，还需要额外的认证，手机邮箱等
- 单点登录SSO：允许用户通过一次登录，访问多个系统或服务。
- 一次性密码(OTP)
- 基于公钥的认证：ssh密钥认证，x.509证书，常用于vpn，以及设备与管理系统通信的场景。

#### 2.1.4 授权

授权是身份验证后决定用户可以访问哪些资源、执行什么操作的关键步骤。
##### 静态授权
- 基于角色的访问控制（RBAC）
- 基于属性的访问控制（ABAC）
- 权限精细化控制：
    1. 对象级控制：控制用户对具体对象的访问权限，如只能查看用户自己的资源
    2. 字段级控制：控制用户对具体字段的访问权限，如不能查看某个记录的敏感字段
    3. 操作级控制：控制用户对资源的具体操作权限，比如Read、Write、Delete、Execute

##### 动态授权
- 基于时间或条件的访问控制：
    - 根据时间段设置访问权限，用户不在有效时间段内无法访问。
    - 条件性访问：基于特定条件对用户进行授权，用户必须满足某些特定条件才能访问。
- 权限的委派与临时权限
    - 权限委派：允许用户将其权限的一部分委派给其他用户，这个典型场景就是代维，将自己账号的权限代维给其他用户，有其他用户登录代维用户，帮其做业务操作。
    - 临时权限：设置某些权限的有效时间范围，用户只能在授权的时间段内访问资源，通常用于临时权限授予或应急。

## 三、如何实现IAM

实现一个iam，可以用开源组件自己组合，也可以选用商业的iam解决方案。下面是一些整理。

### 3.1 iam选型

#### 3.1.1 商业方案
竹云的iam方案:[https://www.bamboocloud.com/](https://www.bamboocloud.com/)
这块不细说，官网有介绍。

#### 3.1.2 开源组件选型

##### 3.1.2.1 Java
1. Spring Security
spring security支持认证、授权，支持OAuth2，SAML2
2. Apache Shiro
Shiro支持认证、授权、会话管理

##### 3.1.2.2 python
1. Flask-Security
支持基于会话、token的认证，注册、角色和权限管理、账号激活、密码管理、双因子认证
2. FastAPI+OAuth2
支持搭配oauth

### 3.2 自研

自研这块，目前微服务是趋势，3A模块，考虑到单一服务职责，可以将iam分为3个模块。
#### 3.2.1 session模块
会话模块，提供会话管理，保存会话信息，会话一般不考虑持久化，在分布式场景下，可以考虑使用redis做存储。

#### 3.2.2 Authentication模块
登录认证模块，包含各种登录功能实现，比如单点登录、用户名密码登录、双因子登录、二维码登录等。这个模块也不涉及持久化存储。

#### 3.2.3 Authorization模块
在这个模块将用户管理也放进去，角色模型及权限控制等功能。

#### 3.2.4 SDK
大型系统，3A服务独立，为了方便业务应用接入3A，需要提供sdk实现，业务方只做配置即可，无需编写太多代码逻辑。
sdk主要实现的功能是：
1. 基于HTTP接口的会话有效性验证
2. 基于HTTP接口的统一权限校验
3. 获取当前会话的用户信息
4. 角色和权限以及功能接口的注册

## 四、认证协议

目前应用和API基本都需要认证，授权、和访问控制。现在有很多标准，比如JSON Web Tokens（JWT），OAuth 2.0，OpenID Connect（OIDC）和SAML。

### 4.1 JWT

jwt是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准。
JWT的声明一般被用来在身份提供者和服务者见传递被认证的用户信息，该token也可直接被用于认证或者加密。
[](/img/iam/jwt.png)

#### 4.1.1 传统session认证

http是无状态协议，如果用户通过账号和密码发起一次请求登录。下一次请求还需要在次指定账号密码等信息。因为http协议是无状态的，并不知道是哪个用户发出的请求。
为了解决这个问题，服务器会存储一份用户登录的信息，这个信息会返回给浏览器，存在cookie中。下次http请求调用时，带上这个cookie，则服务器就能识别到是哪个用户的请求。

#### 4.1.2 传统session认证缺点

- 传统session信息需要服务端存储，可以存redis，可以存内存。随着认证用户增多，服务端开销会大。
- CSRF：浏览器会对发起的请求，自动携带对应站点的cookie，这导致可以从恶意站点应用发起一个当前站点应用的请求，从而携带当前站点的cookie到恶意应用后端。

#### 4.1.3 基于token的鉴权机制

token类似于session的产生过程，只不过将token值传递给前端时，前端发起请求是将token字段放到header里。

#### 4.1.4 jwt具体是什么

jwt是一个字符串，由三段信息构成，通过`.`连接。比如
```txt
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
##### 4.1.4.1 jwt构成
第一部分是header，第二部分是payload，第三部分是signature
- header：指定token类型和算法
- payload：包含json对象，用于存储数据
- signature：根据header和payload及secret生成的签名信息

这里面header和payload都是只是通过base64加密的字符串，signature是根据header中的算法以及jwt服务端定义的私钥生成的数字签名。
所以payload里面不要存储敏感数据，因为是公开的。但是这个jwt token也无法篡改，因为签名是基于前两部分生成的。

##### 4.1.4.2 jwt安全性

jwt的安全性就在于私钥，如果私钥泄漏了，客户端就能生成jwt了。

#### 4.1.5 什么是Bearer Token
任何一种通过Authorization: Bearer <token>传输的访问令牌。
JWT是Bearer Token的一种实现。

### 4.2 OAuth 2.0

oauth时一个授权机制，用来授权第三方应用，获取用户数据，而不需要共享用户的凭据（如密码）

#### 4.2.1 工作原理

1. 用户在授权服务器上登录，并授权应用程序访问其资源
2. 授权服务器发送一个访问令牌（Access Token），应用程序使用该令牌访问用户资源。

[](/img/iam/oauth.png)

#### 4.2.2 核心组件

几个核心概念：

- Third-party application：第三方应用程序（可以理解为client）
- Resource Owner：资源所有者
- Authorization server：认证服务器，即服务提供商用来处理认证的服务器
- Resource Server：资源服务器，即服务提供商存放用户生成的资源的服务器。可以跟认证服务器是同一个，也可以区分开。

#### 4.2.3 客户端的授权模式

这里面的客户端，不是指浏览器，而是指你正在访问的应用。举个例子，你想要访问一个beer网站，这个网站要登录才能获取资源，他提供了github的登录入口，点击github登录，就能弹出github的授权界面，然后点击确定，就能将github登录后的用户名信息登录到当前beer网站中了。
这个beer网站就是客户端client。
服务提供商和认证服务器都是github

##### 4.2.3.1 授权码模式
授权码模式是功能最完整、流程最严密的授权模式。步骤如下：
1. 用户访问浏览器客户端，后者将浏览器地址导向认证服务器。
2. 用户在界面上选择是否给予客户端授权
3. 用户给予授权，认证服务器导向客户端事先指定的重定向URI，同时附上一个授权码
4. 客户端收到授权码，附上早先的重定向URI，向认证服务器申请令牌。（这一步在客户端的后端的服务器上完成，对客户不可见）
5. 认证服务器核对授权码和重定向URI，确认无误后，向客户端发送访问令牌（Access Token）和更新令牌（refresh Token）


##### 4.2.3.2 简化模式

简化模式不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过授权码这个步骤。
步骤如下：
1. 客户端将用户导向认证服务器
2. 用户决定是否给予客户端授权
3. 用户给予授权，认证服务器将用户导向客户端指定的重定向URI，并在URI的hash部分包含访问令牌。
4. 浏览器向资源服务器发出请求，其中不包括上一步收到的Hash值
5. 资源服务器返回一个网页，其中包含的代码可以获取Hash值中的令牌。
6. 浏览器执行上一步获得的脚本，提取出令牌
7. 浏览器将令牌发送给客户端

##### 4.2.3.3 密码模式
密码模式（Resource Owner Password Credentials Grant),中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向“服务提供商“索要授权。
在这种模式中，用户把自己的密码给客户端。但是客户端不得存储密码。
步骤如下：
1. 用户向客户端提供用户名和密码
2. 客户端将用户名和密码发送给认证服务器，向后者请求令牌。
3. 认证服务器确认无误后，向客户端提供了令牌。

#### 4.2.4 更新令牌
如果用户访问的时候，客户端的访问令牌已经过期，则需要使用更新令牌申请一个新的访问令牌。
客户端发出更新令牌的HTTP请求，包含以下参数：
- granttype：授权模式，此处固定值refreshToken
- refresh_token: 早前收到的更新令牌，必选项
- scope：表示申请的授权范围，不可以超出上一次申请的范围。


### 4.3 OIDC协议

OpenID Connect（OIDC）是一种基于 OAuth 2.0 的身份认证协议，用于在不同的应用和系统之间实现单点登录（SSO）和身份验证。它允许应用程序从身份提供者（如 Google、GitHub 等）获取用户的身份信息，而无需管理用户的凭据。

#### 4.3.1 ID令牌字段含义
以下面内容举例：
```
{
  "iss": "https://dex.example.com/",
  "sub": "R29vZCBqb2IhIEdpdmUgdXMgYSBzdGFyIG9uIGdpdGh1Yg",
  "aud": [
    "kubernetes",
    "kubeconfig-generator"
  ],
  "exp": 1712945837,
  "iat": 1712945237,
  "azp": "kubeconfig-generator",
  "at_hash": "OamCo8c60Zdj3dVho3Km5oxA",
  "c_hash": "HT04XtwtlUhfHvm7zf19qsGw",
  "email": "maksim.nabokikh@palark.com",
  "email_verified": true,
  "groups": [
    "administrators",
    "developers"
  ],
  "name": "Maksim Nabokikh",
  "preferred_username": "maksim.nabokikh"
}
```
- iss: https://dex.example.com/
这是 Issuer，即签发此令牌的服务器。在这个例子中，签发者是 https://dex.example.com/，表明令牌来自该身份提供者 (Identity Provider, IdP)。
- sub: R29vZCBqb2IhIEdpdmUgdXMgYSBzdGFyIG9uIGdpdGh1Yg
Subject，通常是唯一标识用户的 ID。这是用户的标识符，可能经过了编码处理，但它在身份提供者中唯一地表示这个用户。
- aud: ["kubernetes", "kubeconfig-generator"]
Audience，这个令牌的目标受众，通常是应用的标识符。在这个例子中，令牌是为 kubernetes 和 kubeconfig-generator 这两个客户端生成的。这意味着只有这些客户端可以使用此令牌。
- exp: 1712945837
Expiration Time，表示令牌过期的时间（Unix 时间戳格式）。在此时间之后，令牌将不再有效。
- iat: 1712945237
Issued At，表示令牌签发的时间（Unix 时间戳格式）。
- azp: kubeconfig-generator
Authorized Party，即最终被授权使用此令牌的客户端。在这里是 kubeconfig-generator。
- at_hash: OamCo8c60Zdj3dVho3Km5oxA
Access Token Hash，如果该令牌是在认证和授权请求中产生的，at_hash 是 Access Token 的哈希值，用于验证 Access Token。
- c_hash: HT04XtwtlUhfHvm7zf19qsGw
Code Hash，类似于 at_hash，c_hash 是 Authorization Code 的哈希值，用于验证授权码的完整性。
- email: maksim.nabokikh@palark.com
用户的 电子邮件地址。这是一个自定义的 claim，提供了用户的联系信息。
- email_verified: true
电子邮件是否已经经过验证。true 表示该电子邮件地址已经过验证。
- groups: ["administrators", "developers"]
用户所属的 组，这些组可能决定用户在不同系统中的权限或角色。
- name: Maksim Nabokikh
用户的 全名。这是一个自定义的 claim，用于更好地识别用户。
- preferred_username: maksim.nabokikh
用户的 首选用户名。这通常是在系统中显示给其他人的用户名。



## 参考文档
1. [https://docs.authing.cn/v2/concepts/authentication.html](https://docs.authing.cn/v2/concepts/authentication.html)

2. [https://guptadeepak.com/demystifying-jwt-oauth-oidc-and-saml-a-technical-guide/](https://guptadeepak.com/demystifying-jwt-oauth-oidc-and-saml-a-technical-guide/)

3. [https://ssojet.com/blog/saml-vs-openid-vs-oauth-vs-jwt/](https://ssojet.com/blog/saml-vs-openid-vs-oauth-vs-jwt/)

4. [https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html](https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)