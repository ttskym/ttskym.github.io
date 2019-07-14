---
layout:     post
title:      "TLS 握手"
subtitle:   "TLS Handshake"
date:       2019-05-01 22:40:00
author:     "jk"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - TLS
    - Security
    - Network
    - Cryptography
---

## 前言

HTTPS 或者说 SSL or TLS 现在都是老生常谈的东西了，为什么还要写这篇文章？

TLS 1.3 2018年发布了，在看1.3的变动前有必要盘点下1.2的主要原理。另外，在我查找握手细节的过程中发现很多博客没有把一些细节解释清楚，主要是关于RSA和DH两种密钥协商协议所产生的一些细节上的区别，所以就有了这篇文章。

本篇文章已发布于FreeBuf [TLS握手：回顾1.2、迎接1.3](https://www.freebuf.com/articles/network/202504.html)

## 背景

SSL/TLS 的解决的威胁：

1. 数据泄露问题（Data Leak）：信息加密，明文传输->密文传输
2. 数据篡改（Data tampering）：保证数据完整性， MAC, AE(after TLS 1.2)
3. 身份认证问题（spoofing/pretending）: 身份鉴别，数字证书

HTTPS是什么？可简单的理解为HTTP over SSL

SSL在设计上拥有可扩展性，不仅可以和HTTP结合作HTTPS，还可应用到很多其他的应用成协议，如FTP，Telnet等等。

### 历史

1994 NetScape SSL 1.0 未发布

1995 NetScape SSL 2.0

1996 SSL 3.0

1999 ISOC TLS 1.0（SSL 3.1）

2006 TLS 1.1 (SSL 3.2)

2008 TLS 1.2 (SSL 3.3)

2018 TLS 1.3

## 基础知识

### 非对称加密与对称加密

简单的理解，非对称加密加解密使用不同的密钥，对称加解密使用相同的密钥。

非对称加密性能不如对称加密。

非对称加密私钥要保持隐秘性，公钥是公开的。私钥加密，公钥解密，叫签名，可用来验证身份，公钥加密，私钥解密，就是常见的加密传输信息。

#### 证书和签名

简单的说，

CA 将申请者的公钥和信息进行单向hash，生成摘要，然后CA用自己的私钥对摘要进行签名，最好将申请信息(公钥)和签名整合生成证书。

Client(电脑和浏览器)内置了CA的根证书，根证书中包含CA公钥，用CA公钥解密证书，验证证书，拿到摘要，然后按照同样的HASH算法生成摘要，将两个摘要进行比对。

比较详细的通俗解释参考：[一篇文章读懂HTTPS及其背后的加密原理](https://mp.weixin.qq.com/s/3gI8avaaaEaBJjOKitN7Fw)

### 密钥交换(密钥协商)及身份认证

SSL设计的最重要关键点就是密钥交换。

单纯用对称加密，如果双方不预先内置(PSK)，直接明文传输，机密性无法保证；加密传输，产生先有鸡还是先有蛋的问题。

非对称加密，主要有两种方式：基于RSA的密钥协商和基于DH的密钥协商。

基于RSA的密钥协商，引入了CA证书，可以解决数据泄露问题，CA证书解决身份认证问题。

[基于DH的密钥协商](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)，`通讯双方在完全没有对方任何预先信息的条件下通过不安全信道创建起一个密钥`, 可以解决数据泄露问题，但是不能解决身份认证问题，会存在MITM攻击。所以DH密钥协商需要配合某种签名算法，如果没有配合任何签名算法，称为“DH-ANON”。此外，DH有很多种，DH本身依赖的是求离散对数问题困难性，而变种ECDH依赖求解椭圆曲线离散对数问题困难性。

## SSL/TLS handshake

### Overview

```
      Client                                               Server

      ClientHello                  -------->
                                                      ServerHello
                                                     Certificate*
                                               ServerKeyExchange*
                                              CertificateRequest*
                                   <--------      ServerHelloDone
      Certificate*
      ClientKeyExchange
      CertificateVerify*
      [ChangeCipherSpec]
      Finished                     -------->
                                               [ChangeCipherSpec]
                                   <--------             Finished
      Application Data             <------->     Application Data
```

### Client Hello

包含：

- client 支持的协议版本
- client random value
- 支持的加密套件列表
- ......, 例如：压缩方法

### Server Hello

包含：

- 确认使用的协议版本
- server random value
- 选择的加密套件
- ...

#### 加密套件命名

`TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256`

- `TLS` 协议
- `ECDHE` 密钥交换协议
- `ECDSA` 身份认证算法
- `AES_128_GCM` 对称加密算法
- `SHA256` MAC

### Server Certificate(可选)

服务器证书，这一步紧随在Server Hello 之后，用来供客户端验证服务器的身份。

### Server Key Exchange Message(可选)

      This message will be sent immediately after the server Certificate
      message (or the ServerHello message, if this is an anonymous
      negotiation).
这个步骤在Server Certificate后，按照RFC的说法，如果没有Server Certificate，则在Server Hello之后。

该环节也是可选的。如果秘钥协商算法选用的是诸如RSA这类，则不会发送这部分信息。RFC中的原文是

```
   The ServerKeyExchange message is sent by the server only when the server Certificate message (if sent) does not contain enough data to allow the client to exchange a premaster secret.  
```

可简单的理解，如果是采用的非对称加密算法来进行秘钥协商，可以通过公钥加密发送预主秘钥的算法不需要发送这部分信息。如RSA，而像DH及其变种这种密钥交换算法则需要通过这个环节发送算法所需的参数信息。RFC中给出的DH参数结构参考：

```
      struct {
          opaque dh_p<1..2^16-1>;
          opaque dh_g<1..2^16-1>;
          opaque dh_Ys<1..2^16-1>;
      } ServerDHParams;     /* Ephemeral DH parameters */

      dh_p
         The prime modulus used for the Diffie-Hellman operation.

      dh_g
         The generator used for the Diffie-Hellman operation.

      dh_Ys
```

### Certificate Request(可选)

SS分为单向SSL(One-way SSL)和双向SSL(Two-way SSL)，两者区别主要是server端是否需要对client端进行身份验证，常见场景如银行提供的USB证书。双向SSL，这部分信息会在Server Key Exchange Message之后，单向SSL中，不会发送这部分信息。

### Server Hello Done

最后，server发送这部分信息来结束Sever hello， 等待客户端回应。

### Client Certificate(可选)

这部分信息是Server Hello Done 之后可以发送的第一部分信息，对应Certificate Request，也是可选部分。

### Client Key Exchange Message

这部分信息在Client Certificate之后发送，在单向SSL中，在Server Hello Done后发送。这部分信息包含的最重要的一部分信息是premaster secret(预主密钥)，permaster的说法对应的是RSA，而在DH中对应的是DH exponent。

如果密钥协商算法是RSA， client会生成一个48字节长的预主密钥，然后用server的公钥加密后发送。

如果是DH算法，则发送client端生成的DH public value(dh_Yc)(请参考DH的数学原理), 此外如果之前的client certificate已经加密发送了这个值，那这里也会发送一个空值过去。

### Certificate Verify(可选)

这部分信息如果需要发送，会跟随在Client Key Exchange Message部分后。

### Finished   

在最后，发送完ChangeCipherSpec，验证了密钥交换协议和身份认证成功后，会发送finish信息。至此握手完成。

### Master Secret

无论用哪种密钥协商算法，最后的主密钥计算方法是相同的——TLS采用了一个PRF来做主密钥的计算，PRF的参数就是之前的两个随机数加上预主密钥。

```
      master_secret = PRF(pre_master_secret, "master secret",
                          ClientHello.random + ServerHello.random)
                          [0..47];
```

### Q & A

1. RSA的时候直接设置秘钥不行么，为什么需要引入random

   SSL不信任每个client都能产生安全的随机数，所以client和server各引入了一个random value

2. 三个random怎么生成最终的秘钥

   这个问题，很多帖子都没有说明，最开始看的时候这是我的一个疑惑，其实RFC中给出了很明确的回答，参考握手中的部分。

3. 为什么不直接用非对称加密通信？

   性能问题。因为非对称加密的性能不如对称加密，所以TLS的核心任务就是怎么产生一个双方共同的、秘密的、可信密钥。

## TLS 1.3

TLS 1.3 废除了RSA 密钥协商机制，保证PFS，PFS主要是为了避免回溯性破解，回溯性破解简单的说就是如果在将来拿到了双防的私钥，能推导出会话密钥，就能解密通信内容。而RSA的私钥是稳定长期不变的，所以无法保证PFS，而DH可以强制每次生成随机私钥，之后强制销毁，可以保证PFS，这种DH变种叫做DHE。



TLS1.3废除了HMAC(SHA1, MD5),只采用AEAD验证完整性，AEAD是AE的变种，可以在解密的同时验证完整性



TLS1.3中，对称加密算法只保留了AES。压缩特性废除。



TLS1.3由于废除了RSA的密钥协商机制，所以握手协议的过程变动较大，比如DH算法的参数1.2前的版本由server生成，整个握手过程2-RTT，而1.3中，DH参数直接在第一步由client生成，整个握手过程1-RTT。



详细的TLS1.3握手过程参考[RFC 8446](https://tools.ietf.org/html/rfc8446#page-133)，有空再写。



【Ref】：

[TLS Security 5: Establishing a TLS Connection](https://www.acunetix.com/blog/articles/establishing-tls-ssl-connection-part-5/)

[SSL/TLS协议运行机制的概述](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)

[SSL Handshake explained](https://medium.com/@kasunpdh/ssl-handshake-explained-4dabb87cdce)

[一篇文章读懂HTTPS及其背后的加密原理](https://mp.weixin.qq.com/s/3gI8avaaaEaBJjOKitN7Fw)

[SSL/TLS 握手过程详解](https://www.jianshu.com/p/7158568e4867)