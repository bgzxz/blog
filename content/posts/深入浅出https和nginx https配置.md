---
title: "深入浅出https和nginx https配置"
date: 2019-06-01T08:11:54+08:00
draft: true
---
# 为什么需要https

互联网的早期设计很少考虑安全，核心协议基本都是不安全的，只能依靠所有的参与方的诚信行为来保证安全。早期的互联网的只有少数的几个节点构成，那时这种方式也许可行。但是随着全球互联网的飞速的发展，我们的日常生活中的方方面面都在使用互联网，比如在线购物，在线交易，实时聊天等。这些日常使用的在线服务很多都涉及用户的敏感数据和隐私数据以及资金账户数据，而这些数据基本都是通过广泛使用的http协议来传输的，因此实现一种安全的http协议就非常的有价值和重要。

# https是什么

由于网络协议是通过分层结构来组织的，提供了很好的扩展性，因此在设计安全的http时就可以不用重新设计协议，而是利用网络协议的分层结构，在http协议的下一层来解决安全传输问题，这样不仅可以解决http协议的安全问题，同时也可以解决其他应用层协议的安全问题，从而实现一套通用的安全传输层，由此TLS协议就诞生了（前身SSL）。因此https可以解释为HTTP over TLS。

# TLS协议

#### 目标
* 加密安全
 
    为任意交换数据的双方提供隐私和数据完成性以及身份认证

* 互操作性

    独立的开发人员能够使用通用的加密参数开发程序和库，使它们可以相互通信
* 可扩展性

    高效开发和部署加密协议，以及扩展协议
* 效率

    性能成本在可接受范围

#### TLS在OSI模型中的位置

![osi](/images/osi.jpg)

#### 协议历史

![tls_history](/images/tls_history.png)

本文将重点介绍这1.3和1.2两个版本

#### TLS中的密码学

由于TLS是在现代密码学的基础上构建出来的，因此在了解TLS之前，必须对密码学的基础知识有一定的了解。

安全的三个核心需求为：保持秘密（机密性）、验证身份（真实性），保证传输安全（完整性）。而要解决这些核心需求，必须通过组合各种加密基元来实现相应的功能。

* 对称加密

      对称加密又称私钥加密，是一种混淆算法，能够让数据在非安全信道上进行安全通信。

    ![对称加密](/images/symmetric_encryption.jpg)

* 散列函数

    散列函数是将任意长度的输入转化为定长输出的算法。散列函数的结果经常被简称为散列（Hash）。密码学散列函数必须具备以下特征：
    * 抗原像性（单向性）

        给定一个散列，计算上无法找到或者构造出生成它的消息
    * 抗第二原像性(弱抗碰撞性)

        给定一条消息和它的散列，计算上无法找到一条不同的消息具有相同的散列
    * 强抗碰撞性

        计算上无法找到两条散列相同的消息
* 消息验证码

    散列函数可以用于验证数据完整性，但仅在数据的散列和数据本身分开传输的条件下如此。消息验证码（MAC）或者使用密钥的散列是以身份验证扩展了散列函数的密码学函数。只有拥有散列密钥，才能生成合法的MAC。

* 分组密码模式

    分组密码模式是为了加密任意长度的数据而设计的密码学方案，是对称密码的扩展。所有分组密码模式都支持机密性，不过有些将其与身份验证联系起来。

* 非对称加密

    非对称加密又称公约加密，它是另一种方法，使用两个密钥，而不是一个；其中一个密钥是私密的，另一个是公开的。公钥加密的数据只能用私钥才能解密。私钥加密的数据任何人都可以用公钥解密。后者不提供机密性，但是可以用作数字签名。

    ![非对称加密](/images/asymmetic_encryption.jpg)

* 数字签名

    数字签名（又称公钥数字签名）是一种类似写在纸上的普通的物理签名，但是使用了公钥加密领域的技术实现，用于鉴别数字信息的方法。 一套数字签名通常定义两种互补的运算，一个用于签名，另一个用于验证。 数字签名，就是只有信息的发送者才能产生的别人无法伪造的一段数字串，这段数字串同时也是对信息的发送者发送信息真实性的一个有效证明。

    ![数字签名](/images/digital_signature.png)

* 随机数生成

    在密码学中，所有的安全性都依赖于生成随机数的质量。

只有将这些加密基元组合成方案和协议才能满足复杂的安全需要，TLS就是这样的一个协议。

# TLS协议

#### 纪录协议

TLS以纪录协议实现。纪录协议负责在传输连接上交换的所有底层消息，并配置加密。

![tls纪录协议](/images/tls_record.jpg)

TLS主要包含四个核心子协议：握手协议，密钥规格变更协议，应用数据协议，警报协议。下面主要介绍握手协议。

* 握手协议

    #### [TLS 1.2](https://tools.ietf.org/html/rfc5246) 

    * 完整握手

         ![tls1.2-full](/images/tls1.2-full.jpg)

    * 简短握手

        * [会话恢复](https://tools.ietf.org/html/rfc5246)

            ![会话恢复](/images/tls1.2-resume-session.jpg)
        
        * [ticket恢复](https://tools.ietf.org/html/rfc5077)

            ![ticket恢复](/images/tls1.2-ticket.jpg)

    #### [TLS 1.3](https://tlswg.org/tls13-spec/draft-ietf-tls-tls13.html)

    * 完整握手

        ![完整握手](/images/tls1.3-full.png)
    * PSK

        ![psk](/images/tls1.3-psk.png)
    * 0-RTT Data

        ![0rtt](/images/tls1.3-0rtt.png)

#### 密钥交换

在TLS中，会话安全性取决于称为主密钥的48字节共享密钥。密钥交换的目的是为了计算预主密钥，从而生成主密钥。

主要的密钥交换算法：

* RSA

    客户端生成预主密钥，并以服务器的公钥加密传送给服务器。被动攻击者只要获取私钥就可以解密所有加密数据。正在被支持前向保密的算法替代。
* DHE_RSA

    使用DHE协商密钥和RSA进行身份验证。支持前向保密，缺点执行缓慢。
* ECDHE_RSA

    ECDHE和RSA结合的交换算法。支持前向保密，速度快。
* ECDHE_ECDSA

    ECDHE和ECDSA结合的交换算法。支持前向保密，速度快。

DH算法

![DH](/images/dhe.png)

#### 密码套件

* tls1.2
套件组成

 * 身份验证方法
 * 密钥交换方法
 * 加密算法
 * 加密密钥大小
 * 密码模式
 * MAC算法
 * PRF
 * 用于Finished消息的散列函数(TLS1.2)
 * verify_data结构的长度

    {{< highlight bash >}}
ECDHE-ECDSA-AES256-GCM-SHA384 TLSv1.2 Kx=ECDH     Au=ECDSA Enc=AESGCM(256) Mac=AEAD
ECDHE-RSA-AES256-GCM-SHA384 TLSv1.2 Kx=ECDH     Au=RSA  Enc=AESGCM(256) Mac=AEAD
DHE-RSA-AES256-GCM-SHA384 TLSv1.2 Kx=DH       Au=RSA  Enc=AESGCM(256) Mac=AEAD
ECDHE-ECDSA-CHACHA20-POLY1305 TLSv1.2 Kx=ECDH     Au=ECDSA Enc=CHACHA20/POLY1305(256) Mac=AEAD
ECDHE-RSA-CHACHA20-POLY1305 TLSv1.2 Kx=ECDH     Au=RSA  Enc=CHACHA20/POLY1305(256) Mac=AEAD
DHE-RSA-CHACHA20-POLY1305 TLSv1.2 Kx=DH       Au=RSA  Enc=CHACHA20/POLY1305(256) Mac=AEAD
ECDHE-ECDSA-AES128-GCM-SHA256 TLSv1.2 Kx=ECDH     Au=ECDSA Enc=AESGCM(128) Mac=AEAD
ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 Kx=ECDH     Au=RSA  Enc=AESGCM(128) Mac=AEAD
DHE-RSA-AES128-GCM-SHA256 TLSv1.2 Kx=DH       Au=RSA  Enc=AESGCM(128) Mac=AEAD
ECDHE-ECDSA-AES256-SHA384 TLSv1.2 Kx=ECDH     Au=ECDSA Enc=AES(256)  Mac=SHA384
ECDHE-RSA-AES256-SHA384 TLSv1.2 Kx=ECDH     Au=RSA  Enc=AES(256)  Mac=SHA384
DHE-RSA-AES256-SHA256   TLSv1.2 Kx=DH       Au=RSA  Enc=AES(256)  Mac=SHA256
ECDHE-ECDSA-AES128-SHA256 TLSv1.2 Kx=ECDH     Au=ECDSA Enc=AES(128)  Mac=SHA256
ECDHE-RSA-AES128-SHA256 TLSv1.2 Kx=ECDH     Au=RSA  Enc=AES(128)  Mac=SHA256
DHE-RSA-AES128-SHA256   TLSv1.2 Kx=DH       Au=RSA  Enc=AES(128)  Mac=SHA256
AES256-GCM-SHA384       TLSv1.2 Kx=RSA      Au=RSA  Enc=AESGCM(256) Mac=AEAD
AES128-GCM-SHA256       TLSv1.2 Kx=RSA      Au=RSA  Enc=AESGCM(128) Mac=AEAD
AES256-SHA256           TLSv1.2 Kx=RSA      Au=RSA  Enc=AES(256)  Mac=SHA256
AES128-SHA256           TLSv1.2 Kx=RSA      Au=RSA  Enc=AES(128)  Mac=SHA256
{{< /highlight >}}

* tls1.3
{{< highlight bash >}}
TLS_AES_256_GCM_SHA384  TLSv1.3 Kx=any      Au=any  Enc=AESGCM(256) Mac=AEAD
TLS_CHACHA20_POLY1305_SHA256 TLSv1.3 Kx=any      Au=any  Enc=CHACHA20/POLY1305(256) Mac=AEAD
TLS_AES_128_GCM_SHA256  TLSv1.3 Kx=any      Au=any  Enc=AESGCM(128) Mac=AEAD
{{< /highlight >}}

#### TLS1.3和1.2的区别

 * 握手时间更短
 * 只支持AEAD对称加密算法
 * 添加0-RTT模式
 * 删除静态RSA和Diffie-Hellman密码套件
 * 所有ServerHello之后的消息都是加密的，更加的安全

# nginx https配置

* 通过源码安装nginx
{{< highlight bash >}}
sudo wget http://nginx.org/download/nginx-1.17.0.tar.gz
sudo tar -xzf nginx-1.17.0.tar.gz
sudo wget https://www.openssl.org/source/openssl-1.1.1c.tar.gz
sudo tar -xzf openssl-1.1.1c.tar.gz
sudo cd nginx-1.17.0
sudo ./configure --with-http_ssl_module --with-http_v2_module --with-openssl=../openssl-1.1.1c
sudo make & make install
{{< /highlight >}}

* 使用certbot申请[letsencrypt证书](https://letsencrypt.org/)
{{< highlight bash >}}
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot 
＃生成rsa证书
certbot certonly -d *.docbot.dev --manual
＃生成ecdsa证书

openssl ecparam -genkey -name prime256v1 > ecdsa.key

openssl req -new -sha256 -key ecdsa.key -out ecdsa.csr

certonly -d *.docbot.dev --csr ecdsa.csr --manual

openssl ec -out docbot.dev.ecdsa.prikey.pem < ecdsa.key

{{< /highlight >}}

* 配置nginx开启双证书
{{< highlight bash >}}
    ssl_session_cache shared:SSL:10m;  # 10MB -> ~40,000 sessions.
    ssl_session_timeout 24h;           # 24 hours
    ssl_buffer_size 1400;              # 1400 bytes to fit in one MTU
    server {
        listen       443 ssl http2;
        server_name  *.docbot.dev;
        access_log  logs/ssl_access.log  ssllog if=$logme;
        ssl_certificate      docbot.dev.pem;
        ssl_certificate_key  docbot.dev.privkey.pem;
        ssl_certificate      docbot.dev.ecdsa.pem;
        ssl_certificate_key  docbot.dev.ecdsa.prikey.pem;
        ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+ECDSA+AES128:EECDH+aRSA+AES128:RSA+AES128:EECDH+ECDSA+AES256:EECDH+aRSA+AES256:RSA+AES256:EECDH+ECDSA+3DES:EECDH+aRSA+3DES:RSA+3DES:!MD5;
        ssl_prefer_server_ciphers   on;
        ssl_ecdh_curve auto;
        ssl_session_tickets on;
        ssl_session_ticket_key ticket.key;
        ssl_stapling on;
        ssl_stapling_file docbot.dev.stapling;
        ssl_dhparam dhparam.pem;
        add_header Strict-Transport-Security 'max-age=31536000; includeSubDomains';
        keepalive_timeout 300;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }

{{< /highlight >}}

* 通过配置本地hosts验证配置
    ![验证配置](/images/https.png)

# 参考资料

[https权威指南](https://book.douban.com/subject/10746113/)

[nginx官方配置文档](http://nginx.org/en/docs/http/ngx_http_ssl_module.html)




    








