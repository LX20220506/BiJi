# Identity Server

[TOC]



## OAuth 2.0 协议

OAuth:一个关于授权（authorization）的开放网络标准，目前版本是2.0版。

 

### 为何要使用OAuth协议呢？OAuth协议的应用场景

第三方服务方提供服务，某些服务需要用户的同意才能够做到，好比客厅要装修，需要得到主人的同意，拿到钥匙，才能装修，提供服务。

传统做法：

把所有钥匙（账号密码）给工人。但这样，工人可能用这个钥匙开卧室的门。甚至打一个新的钥匙。

缺点：（不安全）

1）服务提供方只是提供一个服务，为了保证服务能提供，就会保存账号密码以供下次提供，这显然不安全。

2）服务提供方有了账号密码，就拥有了用户所有的权利，用户没办法限制服务提供方获得权限的范围和有效期。

3）用户只有修改密码，才能收回赋给服务方的权力。这样做，会使得其他所有获得用户授权的第三方全部失效。

4）只要有一个第三方应用程序被破解，就会导致用户密码泄漏，以及所有被密码保护的数据泄漏

 

OAuth可以解决这些问题。

 

### 一些概念名词：

1）Third-party application：第三方应用程序（client）。

2）HTTP service：HTTP服务提供商。

3）Resource Owner：资源所有者-"用户"（user）。

4）User Agent：用户代理-浏览器。

5）Authorization server：认证服务器，即服务提供商专门用来处理认证的服务器。

6）Resource server：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。

OAuth的作用就是让"客户端"安全可控地获取"用户"的授权，与"服务商提供商"进行互动。

 

### Oauth设计理念

OAuth在"客户端"与"服务提供商"之间，设置一个授权层（authorization layer）。

"客户端"不能直接登录"服务提供商"，只能登录授权层，以此将用户与客户端区分开来。

"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可在登录时，指定授权层令牌的权限范围和有效期。

"客户端"登录授权层以后，"服务提供商"根据令牌的权限范围和有效期，向"客户端"开放用户储存的资料。

 

### OAuth 2.0的运行流程

![](C:\Users\1011751\Desktop\img\20180911171533871.png)


（A）用户打开客户端，客户端要求用户给予授权。

（B）用户同意给予客户端授权。

（C）客户端使用上一步获得的授权（一般是Code），向认证服务器申请令牌TOKEN。

（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。

（E）客户端使用令牌，向资源服务器申请获取资源（用户信息等）。

（F）资源服务器确认令牌无误，同意向客户端开放资源。

 

重点是如何获取Token

客户端获取授权的五种模式：
客户端必须得到用户的授权（authorization grant），才能获得令牌（access token）。

### OAuth 2.0定义了五种授权方式：

授权码模式（authorization code）
简化模式（implicit）
密码模式（resource owner password credentials）
客户端模式（client credentials）
扩展模式（Extension）

#### 1.授权码模式（authorization code）：

功能最完整、流程最严密的授权模式。

特点是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。

![](C:\Users\1011751\Desktop\img\授權碼模式.png)

以微信公众平台公众号网页应用开发流程为例。步骤如下：

（A）用户访问客户端，客户端将用户导向认证服务器。

（B）用户选择是否给予客户端授权。

（C）若用户给予授权，认证服务器将用户导向客户端指定的"重定向URI"（redirection URI），同时附上授权码code。

（D）客户端收到授权码code，附上早先的"重定向URI"，向认证服务器申请token。这一步是在客户端的后台的服务器上完成的，对用户不可见。

（E）认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）

 

一些重要参数：

response_type：表示授权类型，必选项，此处的值固定为"code"
appid：表示客户端的ID，必选项
redirect_uri：表示重定向URI，可选项
scope：表示申请的权限范围，可选项
state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。用于防止恶意攻击

1.引导用户跳转到授权页面：

https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect

参数：

appid      公众号的唯一标识

redirect_uri    授权后重定向的回调链接地址， 请使用 urlEncode 对链接进行处理

response_type      返回类型，请填写code

scope     应用授权作用域，有snsapi_base 、snsapi_userinfo 两种

state       重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节

 

2.通过code获取Token

https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code

参数：

appid      公众号的唯一标识

secret     公众号的appsecret

code       填写获取的code参数（存在有效期，通常设为10分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一一对应关系）

grant_type     填写为authorization_code

 

返回结果：

{
"access_token":"ACCESS_TOKEN", //网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同

"expires_in":7200,  // access_token接口调用凭证超时时间，单位（秒）

"refresh_token":"REFRESH_TOKEN", //用户刷新access_token

"openid":"OPENID",  //用户唯一标识

"scope":"SCOPE"  //用户授权的作用域，使用逗号（,）分隔

 } 

 

access_token：表示访问令牌，必选项。

token_type：表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型。

expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。

refresh_token：表示更新令牌，用来获取下一次的访问令牌，可选项。

scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。

 

#### **2.简化模式（implicit grant type）**

不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过"授权码"这个步骤。

所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。

![](C:\Users\1011751\Desktop\img\簡化模式.png)

步骤如下：

（A）客户端将用户导向认证服务器。

（B）用户决定是否给于客户端授权。

（C）若用户授权，认证服务器将用户导向客户端指定的"重定向URI"，并在URI的Hash部分包含了访问令牌。

（D）浏览器向资源服务器发出请求，其中不包括上一步收到的Hash值。

（E）资源服务器返回一个网页，其中包含的代码可以获取Hash值中的令牌。

（F）浏览器执行上一步获得的脚本，提取出令牌。

（G）浏览器将令牌发给客户端。

 

下面是上面这些步骤所需要的参数。

A步骤中，客户端发出的HTTP请求，包含以下参数：

response_type：表示授权类型，此处的值固定为"token"，必选项。
client_id：表示客户端的ID，必选项。
redirect_uri：表示重定向的URI，可选项。
scope：表示权限范围，可选项。
state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值


例子：

    GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz
    
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
    
    Host: server.example.com

C步骤中，认证服务器回应客户端的URI，包含以下参数：

access_token：表示访问令牌，必选项。
token_type：表示令牌类型，该值大小写不敏感，必选项。
expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。
state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。
例子：

     HTTP/1.1 302 Found
    
     Location: http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA
    
               &state=xyz&token_type=example&expires_in=3600

认证服务器用HTTP头信息的Location栏，指定浏览器重定向的网址。注意，在这个网址的Hash部分包含了令牌。

根据上面的D步骤，下一步浏览器会访问Location指定的网址，但是Hash部分不会发送。接下来的E步骤，服务提供商的资源服务器发送过来的代码，会提取出Hash中的令牌。

 

#### 3.密码模式（Resource Owner Password Credentials Grant）

用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。

在这种模式中，用户必须把自己的密码给客户端，但是客户端不得储存密码。这通常用在用户对客户端高度信任的情况下，比如客户端是操作系统的一部分，或由一个著名公司出品。而认证服务器只有在其他授权模式无法执行的情况下，才能考虑使用这种模式。

![](C:\Users\1011751\Desktop\img\密碼模式.png)

步骤如下：

（A）用户向客户端提供用户名和密码。

（B）客户端将用户名和密码发给认证服务器，向后者请求令牌。

（C）认证服务器确认无误后，向客户端提供访问令牌。

 

B步骤中，客户端发出的HTTP请求，包含以下参数：

grant_type：表示授权类型，此处的值固定为"password"，必选项。
username：表示用户名，必选项。
password：表示用户的密码，必选项。
scope：表示权限范围，可选项。


例子：

     POST /token HTTP/1.1
    
     Host: server.example.com
    
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
    
     Content-Type: application/x-www-form-urlencoded

 


     grant_type=password&username=johndoe&password=A3ddj3w

 


C步骤中，认证服务器向客户端发送访问令牌，例子：

     HTTP/1.1 200 OK
    
     Content-Type: application/json;charset=UTF-8
    
     Cache-Control: no-store
    
     Pragma: no-cache

 


     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
    
       "token_type":"example",
    
       "expires_in":3600,
    
       "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
    
       "example_parameter":"example_value"
    
     }

整个过程中，客户端不得保存用户的密码。

#### 4.客户端模式（Client Credentials Grant）

指客户端以自己的名义，而不以用户的名义，向"服务提供商"进行认证。严格地说，客户端模式并不属于OAuth框架所要解决的问题。在这种模式中，用户直接向客户端注册，客户端以自己的名义要求"服务提供商"提供服务，其实不存在授权问题。

![](C:\Users\1011751\Desktop\img\客戶端模式.png)

步骤如下：

（A）客户端向认证服务器进行身份认证，并要求一个访问令牌。

（B）认证服务器确认无误后，向客户端提供访问令牌。

A步骤中，客户端发出的HTTP请求，包含以下参数：

granttype：表示授权类型，此处的值固定为"clientcredentials"，必选项。
scope：表示权限范围，可选项。


     POST /token HTTP/1.1
    
     Host: server.example.com
    
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
    
     Content-Type: application/x-www-form-urlencoded

 


     grant_type=client_credentials

认证服务器必须以某种方式，验证客户端身份。

B步骤中，认证服务器向客户端发送访问令牌，下面是一个例子。

     HTTP/1.1 200 OK
    
     Content-Type: application/json;charset=UTF-8
    
     Cache-Control: no-store
    
     Pragma: no-cache

 


     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
    
       "token_type":"example",
    
       "expires_in":3600,
    
       "example_parameter":"example_value"
    
     }

 

#### 5.扩展模式（Extension）

扩展模式，是一种自定义模式。规范中仅对“grant type”参数提出了须为URI的要求。对于其他申请数据，可以根据需求进行自定义。

附规范例子。

     POST /token HTTP/1.1
    
     Host: server.example.com
    
     Content-Type: application/x-www-form-urlencoded


     grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Asaml2-
     bearer&assertion=PEFzc2VydGlvbiBJc3N1ZUluc3RhbnQ9IjIwMTEtMDU
     [...omitted for brevity...]aG5TdGF0ZW1lbnQ-PC9Bc3NlcnRpb24-



### 更新令牌

如果用户访问的时候，客户端的"访问令牌"已经过期，则需要使用"更新令牌"申请一个新的访问令牌。

客户端发出更新令牌的HTTP请求，包含以下参数：

granttype：表示使用的授权模式，此处的值固定为"refreshtoken"，必选项。
refresh_token：表示早前收到的更新令牌，必选项。
scope：表示申请的授权范围，不可以超出上一次申请的范围，如果省略该参数，则表示与上一次一致。


例子：

     POST /token HTTP/1.1
    
     Host: server.example.com
    
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
    
     Content-Type: application/x-www-form-urlencoded
     
     grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA

 原文链接：https://blog.csdn.net/qq_34190023/article/details/82629092
