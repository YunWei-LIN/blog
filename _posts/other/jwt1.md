title: jwt (Json Web Tokens)
date: 2018-01-25 21:44:58
tags: jwt
categories: 杂谈
---

__主要内容__
* JWT (Json Web Tokens) 
* Shiro + JWT


*更新历史*
无

JWT 是为分布式，微服务 而生。
这时Web应用是无状态的，即服务器端无状态，就是说服务器端不会存储像session这种东西，而是每次请求时access_token进行资源访问。
如一些REST风格的API， 大型tomcat集群（不做session同步）。
单体WEB应用还是推荐session-cookie机制。


<!-- more -->

***

## JWT 概念
`JWT` 详细说明参照 [jwt](https://jwt.io/)。

下面仅仅罗列`JWT`标准定义的字段（Registered Claim）：

### 载荷（Payload）
+ iss: 该JWT的签发者
+ sub: 该JWT所面向的用户
+ aud: 接收该JWT的一方
+ exp(expires): 什么时候过期，这里是一个Unix时间戳
+ iat(issued at): 在什么时候签发的
+ jti(JWT ID)：该JWT的唯一标识
+ nbf: 这个时间之前token不可用

### 头部（Header）
+ typ: "JWT",
+ alg: 签名或摘要算法， (HS256,HS384,HS512,RS256,RS384,RS512,ES256,ES384,ES512)

### 信息会暴露
在JWT中，不应该在载荷里面加入任何敏感的数据。

### 示例
```sh
Header//头信息
{
    "alg": "HS256",//签名或摘要算法
    "typ": "JWT"//token类型
}
Playload//荷载信息
{
    "iss": "token-server",//签发者
    "exp ": "Mon Nov 13 15:28:41 CST 2017",//过期时间
    "sub ": "wangjie",//用户名
    "aud": "web-server-1"//接收方,
    "nbf": "Mon Nov 13 15:40:12 CST 2017",//这个时间之前token不可用
    "jat": "Mon Nov 13 15:20:41 CST 2017",//签发时间
    "jti": "0023",//令牌id标识
    "claim": {“auth”:”ROLE_ADMIN”}//访问主张
}
Signature//签名信息
签名或摘要算法（
    base64urlencode（Header），
    Base64urlencode（Playload），
    secret-key
）
```

按照JWT规范，对这个令牌定义进行如下操作：
```sh
base64urlencode（Header）
+"."+
base64urlencode（Playload）
+"."+
signature（
    base64urlencode（Header）
    +"."+
    base64urlencode（Playload）
    ,secret-key
）
```


## JWT 撤销（revoke）
https://auth0.com/blog/blacklist-json-web-token-api-keys/

## shiro 集成
http://www.bijishequ.com/detail/578914?p=