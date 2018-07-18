# 苹果证书
![image](http://upload-images.jianshu.io/upload_images/1253942-5aa0f5f1990499b5.jpg)

## 对称加密和非对称加密
* 对称加密 `symmetric cryptography`：加密和解密用的是同一个秘钥
* 非对称加密 `asymmetric cryptography`：加密和解密是不同的钥匙

客户端（Client）和服务器（Server）进行通讯，`Client`和`Server`约定好相同的一把秘钥，Client发送的明文通过这把秘钥进行加密，Server在收到这段加密后的密文后通过事先约定好的那边秘钥进行解密得到明文。理论上只要保证秘钥不被泄露就可以保证安全，但是实际上这种方式很不安全，如果秘钥被破解，又恰好服务器只用了这一个秘钥（这可能是最糟糕的情况），那么服务器和其他的客户端之间的通讯基本上就是完全暴露了。这个例子说的是对称加密。
![对称加密](http://upload-images.jianshu.io/upload_images/1253942-05697d31c8455025.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还有一种情况，比上面的想法更加出色一点。每个端都生成一个“私钥-公钥”对，私钥是自己保管，公钥可以随便分享，用公钥可以解开私钥，用私钥可以解开公钥。例如服务端生成了一个“私钥-公钥”对，自己保留了一份私钥，把公钥给客户端，客户端对发送的消息通过公钥进行加密，服务端在收到这个公钥后用自己的私钥进行解密还原得到明文。

![非对称加密](http://upload-images.jianshu.io/upload_images/1253942-07b9e270cc7088f8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 常见的对称加密有：DES,AES等 
- 常见的非对称加密有：SSL,HTTPS,TLS,RSA。 

秘钥长度越长破解的难度越大。我在iOS持久化数据时用过AES，其他的没有用过。非对称加密中RSA是很有名，被应用的很广泛的数字证书。

## 数字签名和数字证书

看下面的一个场景：

A生成了一对“私钥-公钥”，然后把公钥给了B，B用公钥加密一段消息，发给A，A收到消息后用私钥解密，接着A用私钥加密一段文字给B，B收到后用公钥解密查看明文。如此反复。

上面是一个非对称加密聊天的具体场景，虽然经典，但是还是有一些问题。

```
1.A在收到B的信息的时候不能保证B发的信息是最原始的，即传输的内容有可能被篡改
2.黑客C获取了B的公钥，然后伪装成B和A进行通信

```

上面的情况都是可能出现的，现在这样，A为了保证数据不被改动，先对数据用hash函数生成一个摘要（digest）,然后用私钥对这个摘要进行加密，生成了“数字签名”（signature），B在收到这个消息时先用公钥对摘要进行解密得到摘要并且对数据内容也进行一次hash,比较这次的hash是否与之前解密得到的摘要一致，如果一致，说明数据没有被改动。我们把对内容数据进行hash后再加密生成的一段内容称为“**数字签名**”。

黑客C进行伪装交流的可能性也是有的，为了解决这个办法，A去找了一个权威的“证书中心”（certificate authority,简称CA），为公钥做验证。CA用自己的私钥对A的公钥和一些相关信息一起加密，生成了“**数字证书**”（Digital Certificate）。这样A在进行消息传递时也附带上数字证书，B在收到消息时用CA的公钥解开数字证书拿到真实的公钥，通过这样的方式验证身份。

阮一峰的[数字签名是什么](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)很形象的说明了其中的意思。

## 苹果的证书
在Mac上的钥匙串访问app是专门用来管理证书的。iOS开发者在申请iOS开发证书时，需要通过keychain生成CSR文件（Certificate Signing Request），提交给苹果的`Apple Worldwide Developer Relations Certification Authority(WWDR)`证书认证中心进行签名，最后从苹果官网下载并安装使用。这个过程中还会产生一个私钥，证书和私钥在keychain中得位置如图：
![私钥](http://upload-images.jianshu.io/upload_images/1253942-0814c4c37eb06449.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. iOS系统原本就持有WWDR的公钥，系统首先会对证书内容通过指定的哈希算法计算得到一个信息摘要；
2. 然后使用WWDR的公钥对证书中包含的数字签名解密，从而得到经过WWDR的私钥加密过的信息摘要；
3. 最后对比两个信息摘要，如果内容相同就说明该证书可信。

整个过程如图所示：
![](http://upload-images.jianshu.io/upload_images/1253942-72e507584f9cae02?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在验证了证书是可信的以后，iOS系统就可以获取到证书中包含的开发者的公钥，并使用该公钥来判断代码签名的可用性了

在`objc.io`上面有篇[《Inside Code Signing》](https://www.objc.io/issues/17-security/inside-code-signing/)(中文翻译篇：[代码签名探析](http://objccn.io/issue-17-2/))上详细的讲述了一个已签名应用的组成和一些其他知识

