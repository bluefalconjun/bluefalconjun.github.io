
####Encryption Study

-----
> **对称加密算法**

**`DES,3DES,AES,Blowfish,IDEA,RC5,RC6.`**

特点:
 - 双方使用同样的密钥.
 - 加解密速度快.

-----
> **非对称加密算法**

**`RSA, Rabin, ECC.`**

特点:
 - 分为一组公钥(public)/私钥(private). 合称为Key Pair.
 - 一般用法为使用一个密钥加密, 使用另一个密钥解密.
 - 私钥加密/公钥解密一般用在数字证书签名(CA)中.
 - 公钥加密/私钥解密一般用在双方加密通讯中.

-----
> **单向散列算法**

**`MD1,MD5,SHA1,SHA512,CRC-32`**

特点:
 - 本质是实现信息摘要.
 - 固定输入产生固定输出, 输入的变化导致更大的输出变化.
 - 无法反推原始数据, 输出定长.


-----
> **RSA密钥存储方式**

 - DER
 - PEM

----- 
**DER** 
DER:Distinguished Encoding Rules(可辨别编码规则)，是ASN.1的一种。
 ASN.1:Abstract Syntax Notation One(抽象语法标记)，ASN.1是一种 ISO/ITU-T 标准，描述了一种对数据进行表示、编码、传输和解码的数据格式。它提供了一整套正规的格式用于描述对象的结构，而不管语言上如何执行及这些数据的具体指代，也不用去管到底是什么样的应用程序。
 证书信息一般以二进制的DER格式存储在文件中以供RSA，SSL使用。

-----
**PEM**
DER一般是二进制文件形式存储，打印性较差，因此对DER内容进行base64编码，并补充说明key类型的头和尾就构成了PEM

    -----BEGIN PUBLIC KEY-----
    MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDMYfnvWtC8Id5bPKae5yXSxQTt
    +Zpul6AnnZWfI2TtIarvjHBFUtXRo96y7hoL4VWOPKGCsRqMFDkrbeUjRrx8iL91
    4/srnyf6sh9c8Zk04xEOpK1ypvBz+Ks4uZObtjnnitf0NBGdjMKxveTq+VE7BWUI
    yQjtQ8mbDOsiLLvh7wIDAQAB
    -----END PUBLIC KEY-----

因此PEM,DER实质内容是相同的。

-----
**PKCS＃1**
PKCS＃1结构仅为RSA设计

 - PEM形式

PublicKey
`
-----BEGIN RSA PUBLIC KEY-----
BASE64 ENCODED DATA
-----END RSA PUBLIC KEY-----
`

PrivateKey
`-----BEGIN RSA PRIVATE KEY-----
BASE64 ENCODED DATA
-----END RSA PRIVATE KEY-----`

-----
> **使用openssl生成密钥**

生成2048位RSA密钥,使用3des加密产生密钥文件private.pem
`openssl genrsa -des3 -out private.pem 2048`

导出公钥,默认为PKCS＃8结构
`openssl rsa -in private.pem -outfrom PEM －pubout public.pem`

导出PKCS#1结构的公钥
`openssl rsa -in private.pem -outform DER -RSAPublicKey_out -out public_pcks1.cer`

导出无加密保护的私钥
`openssl rsa -in private.pem -out private_unencrypted.pem -outform PEM`

对issue.clear文件进行3des加密并输出至/out目录, 此处需手动输入密码.
`openssl enc -e -des3 -a - salt -in ./issue.clear -out /out/issue.crypt`

可对加密文件进行解密.
` openssl enc -d -des3 -a -salt -in /out/issue.crypt -out /tmp/issue.new`

使用散列算法提取特征吗
`openssl dgst -md5|-sha1 ./issue.clear`