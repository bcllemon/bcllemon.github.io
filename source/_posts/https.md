title: Nginx+Tomcat 配置 https
date: 2015-10-13 16:13:45
categories: 技术
---
# SSL/TLS简介
安全传输层协议（TLS）与其前辈加密套接字（SSL）都是用于保证 Web 浏览器与 Web 服务器通过安全连接进行通信的技术。利用这些技术，我们所要传送的数据会在一端进行加密，传输到另一端后再进行解密（在处理数据之前）。这是一种双向的操作，服务器和浏览器都能在发送数据前对它们进行加密处理。

SSL/TLS 协议的另一个重要方面是认证。当我们初始通过安全连接与 Web 服务器进行通信时，服务器将提供给 Web 浏览器一组“证书”形式的凭证，用来证明站点的归属方以及站点的具体声明。某些情况下，服务器也会请求浏览器出示证书，来证明作为操作者的“你”所宣称的身份是否属实。这种证书叫做“客户端证书”，但事实上它更多地用于 B2B（企业对企业电子商务）的交易中，而并非针对个人用户。大多数启用了 SSL 协议的 Web 服务器都不需要客户端认证。

# 证书申请
## 使用 OpenSSL 生成 SSL Key 和 CSR

域名，也称为 Common Name，因为特殊的证书不一定是域名：example.com

组织或公司名字（Organization）：Example, Inc.

部门（Department）：可以不填写，这里我们写 Web Security

城市（City）：Beijing

省份（State / Province）：Beijing

国家（Country）：CN

加密强度：2048 位，如果你的机器性能强劲，也可以选择 4096 位

按照以上信息，使用 OpenSSL 生成 key 和 csr 的命令如下

```
openssl req -new -newkey rsa:2048 -sha256 -nodes -out example_com.csr -keyout example_com.key -subj "/C=CN/ST=Beijing/L=Beijing/O=Example Inc./OU=Web Security/CN=example.com"  
```
PS：如果是泛域名证书，则应该填写*.example.com
你可以在系统的任何地方运行这个命令，会自动在当前目录生成 example_com.csr 和 example_com.key 这两个文件
这个 CSR 文件就是你需要提交给 SSL 认证机构的，当你的域名或组织通过验证后，认证机构就会颁发给你一个 example_com.crt
而 example_com.key 是需要用在 Nginx 配置里和 example_com.crt 配合使用的，需要好好保管，千万别泄露给任何第三方。

[b0a12fd3]: http://www.oschina.net/p/java-exportpriv "java-exportpriv"

# Nginx
## 证书处理
Nginx和Tomcat不一样，它要求的是证书文件 .crt 和私钥 .key 。遗憾的是，我们的私钥在keystore里面，而JDK自带的keytool并不提供私钥的导出功能，所以我们得借助第三方工具来导出私钥。
### 1. 导出私钥
有一个开源的私钥导出工具叫做 [java-exportpriv][b0a12fd3] 。它是一个简单的java程序，你下载以后参考它的说明，编译，然后运行即可，非常简单，我就不多罗嗦了。
### 2. 创建 crt
和Apache不一样，Nginx没有Certificat Chain这个参数，所以你要把你的证书和中间证书合并。合并证书很简单，创建一个先的文件 oschina-chain.crt，内容如下：
```
-----BEGIN CERTIFICATE-----
这里是你证书的内容
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
这里是中间证书的内容
-----END CERTIFICATE-----
```
## 修改配置
```
server {
      listen       443 ssl;
      server_name  localhost;
      ssl                  on;
      ssl_certificate      /oschina/webapp/oschina-chain.crt;
      ssl_certificate_key  /oschina/webapp/oschina.key;

  location / {
          include proxy.conf;
          proxy_pass   https://61.145.122.155:443;
      }

  }
```
## 强制 https
```
  server {
      listen 80;
      server_name localhost;
      rewrite ^/(.*) https://$server_name/$1 permanent;
  }
```
## 增强
但是这么做并不安全，默认是 SHA-1 形式，而现在主流的方案应该都避免 SHA-1，为了确保更强的安全性，我们可以采取迪菲－赫尔曼密钥交换

首先，进入 /etc/ssl/certs 目录并生成一个 dhparam.pem
```
openssl dhparam -out dhparam.pem 2048 # 如果你的机器性能足够强大，可以用 4096 位加密  
```

生成完毕后，在 Nginx 的 SSL 配置后面加入
```
        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/ssl/certs/dhparam.pem;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS !RC4";
        keepalive_timeout 70;
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 10m;
```
同时，如果是全站 HTTPS 并且不考虑 HTTP 的话，可以加入 HSTS 告诉你的浏览器本网站全站加密，并且强制用 HTTPS 访问
```
        add_header Strict-Transport-Security max-age=63072000;
        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
```

# Tomcat
## 证书
Tomcat要求的是包含签名过证书的keystore文件和keystore密码。所以我们要先把证书导入keystore
```
$JAVA_HOME/bin/keytool -import -alias oschina -trustcacerts -file oschina.p7s  -keystore oschina.keystore
```
上面的命令中 alias “oschina” 和之前申请证书的时候输入的 alias 要一致。
## 修改Tomcat配置
```
<Connector SSLEnabled="true" acceptCount="100" clientAuth="false"
        disableUploadTimeout="true" enableLookups="false" maxThreads="25"
        port="8443" keystoreFile="/oschina/webapp/oschina.keystore" keystorePass="xxxxxxx"
        protocol="org.apache.coyote.http11.Http11NioProtocol" scheme="https"
        secure="true" sslProtocol="TLS" />
```
