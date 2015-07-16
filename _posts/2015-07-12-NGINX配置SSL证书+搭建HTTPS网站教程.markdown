---
layout: post
title:  "NGINX 配置 SSL 证书 + 搭建 HTTPS 网站教程"
date:   2015-07-12 10:22
categories: nginx
tags: nginx ssl https
---
一、HTTPS 是什么？
-----------------

根据维基百科的解释：

    超文本传输安全协议（缩写：HTTPS，英语：Hypertext Transfer Protocol Secure）是超文本
    传输协议和SSL/TLS的组合，用以提供加密通讯及对网络服务器身份的鉴定。HTTPS连接经常被用于万
    维网上的交易支付和企业信息系统中敏感信息的传输。HTTPS不应与在RFC 2660中定义的安全超文本
    传输协议（S-HTTP）相混。

HTTPS 目前已经是所有注重隐私和安全的网站的首选，随着技术的不断发展，HTTPS 网站已不再是大型网站的专利，所有普通的个人站长和博客均可以自己动手搭建一个安全的加密的网站。

如果一个网站没有加密，那么你的所有帐号密码都是明文传输。可想而知，如果涉及到隐私和金融问题，不加密的传输是多么可怕的一件事。

鉴于本博客的读者都是接近专业人士，我们不再多费口舌，直接进入正题吧。

二、使用 OpenSSL 生成 SSL Key 和 CSR
-------------------------------------

由于只有浏览器或者系统信赖的 CA 才可以让所有的访问者通畅的访问你的加密网站，而不是出现证书错误的提示。所以我们跳过自签证书的步骤，直接开始签署第三方可信任的 SSL 证书吧。

OpenSSL 在 Linux、OS X 等常规的系统下默认都安装了，因为一些安全问题，一般现在的第三方 SSL 证书签发机构都要求起码 2048 位的 RSA 加密的私钥。

同时，普通的 SSL 证书认证分两种形式，一种是 DV（Domain Validated），还有一种是 OV （Organization Validated），前者只需要验证域名，后者需要验证你的组织或公司，在安全性方面，肯定是后者要好。

无论你用 DV 还是 OV 生成私钥，都需要填写一些基本信息，这里我们假设如下：

域名，也称为 Common Name，因为特殊的证书不一定是域名：`example.com`

组织或公司名字（Organization）：`Example, Inc.`

部门（Department）：可以不填写，这里我们写 `Web Security`

城市（City）：`Beijing`

省份（State / Province）：`Beijing`

国家（Country）：`CN`

加密强度：2048 位，如果你的机器性能强劲，也可以选择 4096 位

按照以上信息，使用 OpenSSL 生成 key 和 csr 的命令如下


    openssl req -new -newkey rsa:2048 -sha256 -nodes -out example_com.csr -keyout example_com.key -subj "/C=CN/ST=Beijing/L=Beijing/O=Example Inc./OU=Web Security/CN=example.com"


PS：如果是泛域名证书，则应该填写 `*.example.com`

你可以在系统的任何地方运行这个命令，会自动在当前目录生成 `example_com.csr` 和 `example_com.key`这两个文件

接下来你可以查看一下 `example_com.csr`，得到类似这么一长串的文字

    -----BEGIN CERTIFICATE REQUEST-----
    MIICujCCAaICAQAwdTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaWppbmcxEDAO
    BgNVBAcTB0JlaWppbmcxFTATBgNVBAoTDEV4YW1wbGUgSW5jLjEVMBMGA1UECxMM
    V2ViIFNlY3VyaXR5MRQwEgYDVQQDEwtleGFtcGxlLmNvbTCCASIwDQYJKoZIhvcN
    AQEBBQADggEPADCCAQoCggEBAPME+nvVCdGN9VWn+vp7JkMoOdpOurYMPvclIbsI
    iD7mGN982Ocl22O9wCV/4tL6DpTcXfNX+eWd7CNEKT4i+JYGqllqP3/CojhkemiY
    SF3jwncvP6VoST/HsZeMyNB71XwYnxFCGqSyE3QjxmQ9ae38H2LIpCllfd1l7iVp
    AX4i2+HvGTHFzb0XnmMLzq4HyVuEIMoYwiZX8hq+kwEAhKpBdfawkOcIRkbOlFew
    SEjLyHY+nruXutmQx1d7lzZCxut5Sm5At9al0bf5FOaaJylTEwNEpFkP3L29GtoU
    qg1t9Q8WufIfK9vXqQqwg8J1muK7kksnbYcoPnNgPx36kZsCAwEAAaAAMA0GCSqG
    SIb3DQEBBQUAA4IBAQCHgIuhpcgrsNwDuW6731/DeVwq2x3ZRqRBuj9/M8oONQen
    1QIacBifEMr+Ma+C+wIpt3bHvtXEF8cCAJAR9sQ4Svy7M0w25DwrwaWIjxcf/J8U
    audL/029CkAuewFCdBILTRAAeDqxsAsUyiBIGTIT+uqi+EpGG4OlyKK/MF13FxDj
    /oKyrSJDtp1Xr9R7iqGCs/Zl5qWmDaLN7/qxBK6vX2R/HLhOK0aKi1ZQ4cZeP7Mr
    8EzjDIAko87Nb/aIsFyKrt6Ze3jOF0/vnnpw7pMvhq+folWdTVXddjd9Dpr2x1nc
    y5hnop4k6kVRXDjQ4OTduQq4P+SzU4hb41GIQEz4
    -----END CERTIFICATE REQUEST-----

这个 CSR 文件就是你需要提交给 SSL 认证机构的，当你的域名或组织通过验证后，认证机构就会颁发给你一个 `example_com.crt`

而 `example_com.key` 是需要用在 Nginx 配置里和 `example_com.crt` 配合使用的，需要好好保管，千万别泄露给任何第三方。

三、Nginx 配置 HTTPS 网站以及增加安全的配置
-------------------------------------------

前面已经提到，你需要提交 CSR 文件给第三方 SSL 认证机构，通过认证后，他们会颁发给你一个 CRT 文件，我们命名为 `example_com.crt`

同时，为了统一，你可以把这三个文件都移动到 `/etc/ssl/private/` 目录。

然后可以修改 Nginx 配置文件

    server {
        listen 80;
        listen [::]:80 ssl ipv6only=on;
        listen 443 ssl;
        listen [::]:443 ssl ipv6only=on;
        server_name example.com;

        ssl on;
        ssl_certificate /etc/ssl/private/example_com.crt;
        ssl_certificate_key /etc/ssl/private/example_com.key;
    }

检测配置文件没问题后重新读取 Nginx 即可

nginx -t && nginx -s reload

但是这么做并不安全，默认是 SHA-1 形式，而现在主流的方案应该都避免 SHA-1，为了确保更强的安全性，我们可以采取迪菲－赫尔曼密钥交换

首先，进入 /etc/ssl/certs 目录并生成一个 dhparam.pem

    cd /etc/ssl/certs
    openssl dhparam -out dhparam.pem 2048 # 如果你的机器性能足够强大，可以用 4096 位加密

生成完毕后，在 Nginx 的 SSL 配置后面加入

    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS !RC4";
    keepalive_timeout 70;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

同时，如果是全站 HTTPS 并且不考虑 HTTP 的话，可以加入 HSTS 告诉你的浏览器本网站全站加密，并且强制用 HTTPS 访问

    add_header Strict-Transport-Security max-age=63072000;
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;

同时也可以单独开一个 Nginx 配置，把 HTTP 的访问请求都用 301 跳转到 HTTPS

    server {
            listen 80;
            listen [::]:80 ssl ipv6only=on;
            server_name     example.com;
            return 301 https://example.com$request_uri;
    }


四、可靠的第三方 SSL 签发机构
------------------------------

众所周知，前段时间某 NIC 机构爆出过针对 Google 域名的证书签发的丑闻，所以可见选择一家靠谱的第三方 SSL 签发机构是多么的重要。

目前一般市面上针对中小站长和企业的 SSL 证书颁发机构有：

`StartSSL`

Comodo / 子品牌 Positive SSL

GlobalSign / 子品牌 AlphaSSL

GeoTrust / 子品牌 RapidSSL

其中 Postivie SSL、AlphaSSL、RapidSSL 等都是子品牌，一般都是三级四级证书，所以你会需要增加 CA 证书链到你的 CRT 文件里。

以 Comodo Positive SSL 为例，需要串联 CA 证书，假设你的域名是 example.com

那么，串联的命令是

    cat example_com.crt COMODORSADomainValidationSecureServerCA.crt COMODORSAAddTrustCA.crt AddTrustExternalCARoot.crt > example_com.signed.crt

在 Nginx 配置里使用 example_com.signed.crt 即可

如果是一般常见的 AplhaSSL 泛域名证书，他们是不会发给你 CA 证书链的，那么在你的 CRT 文件后面需要加入 AlphaSSL 的 CA 证书链

AlphaSSL Intermediate CA

五、针对企业的 EV SSL
----------------------

EV SSL，是 Extended Validation 的简称，更注重于对企业网站的安全保护以及严格的认证。

最明显的区别就是，通常 EV SSL 显示都是绿色的条，比如本站的 SSL 证书就是 EV SSL。

如果贵公司想获取专业的 EV SSL，可以随时联系我们 info at cat dot net

六、本文参考文献
----------------

[Apache + WordPress + SSL 完全指南](http://ttt.tt/9/)

[OpenSSL CSR Creation](https://www.digicert.com/easy-csr/openssl.htm)

[NGINX - PhoenixWiki](https://wiki.phoenixlzx.com/page/NGINX/)


## 原文地址
[NGINX 配置 SSL 证书 + 搭建 HTTPS 网站教程 - S.HOW](https://s.how/nginx-ssl/)

