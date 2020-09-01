layout: post
title: "Python Java AES"
date: "2015-10-29 17:16"
categories: 技术
---
# AES
密码学中的高级加密标准（Advanced Encryption Standard，AES），又称高级加密标准Rijndael加密法，
是美国联邦政府采用的一种区块加密标准。这个标准用来替代原先的DES，已经被多方分析且广为全世界
所使用。经过五年的甄选流程，高级加密标准由美国国家标准与技术研究院 （NIST）于2001年11月26日
发布于FIPS PUB197，并在2002年5月26日成为有效的标准。2006年，高级加密标准已然成为对称密钥加密
中最流行的算法之一。该算法为比利时密码学家Joan Daemen和VincentRijmen所设计，结合两位作者的名
字，以Rijndael之命名之，投稿高级加密标准的甄选流程。（Rijdael的发音近于 "Rhinedoll"。）
## 加密填充模式
```
算法/模式/填充                16字节加密后数据长度        不满16字节加密后长度
AES/CBC/NoPadding             16                          不支持
AES/CBC/PKCS5Padding          32                          16
AES/CBC/ISO10126Padding       32                          16
AES/CFB/NoPadding             16                          原始数据长度
AES/CFB/PKCS5Padding          32                          16
AES/CFB/ISO10126Padding       32                          16
AES/ECB/NoPadding             16                          不支持
AES/ECB/PKCS5Padding          32                          16
AES/ECB/ISO10126Padding       32                          16
AES/OFB/NoPadding             16                          原始数据长度
AES/OFB/PKCS5Padding          32                          16
AES/OFB/ISO10126Padding       32                          16
AES/PCBC/NoPadding            16                          不支持
AES/PCBC/PKCS5Padding         32                          16
AES/PCBC/ISO10126Padding      32                          16

```

# ECB(Electronic Code Book电子密码本)模式
ECB模式是最早采用和最简单的模式，它将加密的数据分成若干组，每组的大小跟加密密钥长度相同，然后每组都用相同的密钥进行加密。
**优点:**
- 模式操作简单；
- 有利于并行计算；
- 误差不会被传送；

**缺点:**
- 不能隐藏明文的模式，明文中的重复内容在密文中暴露出来；
- 可能对明文进行主动攻击；

因此，此模式适于加密小消息。
## Java
```
String key = "1234567890123456";

SecretKey secretKey = new SecretKeySpec(key.getBytes(), "AES");
Cipher cipher = Cipher.getInstance("AES");
String data = "abcd";
cipher.init(Cipher.ENCRYPT_MODE, secretKey);
byte[] encrypted = cipher.doFinal(data.getBytes());
System.out.println(Encodes.encodeBase64(encrypted));

cipher.init(Cipher.DECRYPT_MODE, secretKey);
System.out.println(new String(cipher.doFinal(encrypted)));
```
## Python
```
from Crypto.Cipher import AES
from itsdangerous import base64_encode,base64_decode

BS = AES.block_size
pad = lambda s: s + (BS - len(s) % BS) * chr(BS - len(s) % BS)
unpad = lambda s : s[0:-ord(s[-1])]

# key = os.urandom(16) # the length can be (16, 24, 32)
key = b'1234567890123456'
text = 'abcd'
cipher = AES.new(key)
print base64_encode(cipher.encrypt(pad(text)))
print cipher.decrypt(base64_decode(base64_encode(cipher.encrypt(pad(text)))))
```
# CBC(Cipher Block Chaining，加密块链)模式
为了克服ECB模式的安全缺陷，设计了密码分组链接模式，它使得当同一个明文分组重复出现时产生不同的密文分组。对每个分组使用相同的密钥，加密函数的输入是当前的明文分组和前一个密文分组的异或。从效果上看，将明文分组序列的处理连接起来了。
为了产生第一个密文分组，要使用一个初始向量IV，IV必须被发送方和接收方都知道，为了做到最大程度的安全性，IV应该和密钥一样受到保护。
CBC模式的加加解密，要求 key 和 IV 都必须要一致
**优点：**
- 不容易主动攻击,安全性好于ECB,适合传输长度长的报文,是SSL、IPSec的标准。

**缺点：**
- 不利于并行计算；
- 误差传递；
- 需要初始化向量IV

## Java
```
String iv="1234567890123456";
String key = "1234567890123456";

SecretKey secretKey = new SecretKeySpec(key.getBytes(), "AES");
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
IvParameterSpec ivspec = new IvParameterSpec(iv.getBytes());
String data = "abcd";
int blockSize = cipher.getBlockSize();
byte[] dataBytes = data.getBytes();
int plaintextLength = dataBytes.length;
if (plaintextLength % blockSize != 0) {
    plaintextLength = plaintextLength + (blockSize - (plaintextLength % blockSize));
}

byte[] plaintext = new byte[plaintextLength];
System.arraycopy(dataBytes, 0, plaintext, 0, dataBytes.length);
cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivspec);
byte[] encrypted = cipher.doFinal(plaintext);
System.out.println(Encodes.encodeBase64(encrypted));

cipher.init(Cipher.DECRYPT_MODE, secretKey,ivspec);
System.out.println(new String(cipher.doFinal(encrypted)));
```
## Python
```
from Crypto.Cipher import AES
import os
from itsdangerous import base64_encode, base64_decode

BS = AES.block_size
pad = lambda s: s + (BS - len(s) % BS) * chr(BS - len(s) % BS)
unpad = lambda s: s[0:-ord(s[-1])]

# key = os.urandom(16) # the length can be (16, 24, 32)
key = b'1234567890123456'
text = 'abcd'
IV = b'1234567890123456'
cipher = AES.new(key, AES.MODE_CBC, IV=IV)

data = base64_encode(cipher.encrypt(pad(text)))

print data
#
cipher = AES.new(key, AES.MODE_CBC, IV=IV)
print cipher.decrypt(base64_decode(data))
```
