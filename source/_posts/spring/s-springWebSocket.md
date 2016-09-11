title: Spring Web Socket
date: 2015-08-18 19:37:37
tags: Spring WebSocket
categories: spring
---
## 使用情景
Spring Framework是基于STOMP提供的WebSocket服务，适合与对时间延迟非常敏感并且需要高频得交换大量数据的场合。比如但不限于这样的应用：金融，游戏，在线合作等。REST和HTTP API可以和WebSocket混合使用。 Spring Framework允许`@Controller`和`@RestController`同时有HTTP request操作和WebSocket消息操作。