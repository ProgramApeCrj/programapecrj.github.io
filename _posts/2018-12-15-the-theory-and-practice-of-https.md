---
layout: default_post
title : 从理论到实践理解 https
category: web 开发
---



前言：最近发现谷歌浏览器没办法访问自己的 github 博客，其他浏览器都正常。看了一下发现现在谷歌要用 https 才能访问。看来已经要进入全员 https 时代咩。

#### 导航
1. 不安全的网络
1. 对称加密 和 非对称加密
1. 什么是 hash 函数（算法）
1. 数字签名
1. 常见误区
1. 关于 jwt 
1. 证书
1. 实际操作
1. 其他
1. 总结

<br>
# 1. 不安全的网络

网络上信息传输是不安全的。这点就是为什么要有这么多加密的原因。

发送一句话，可能经过多个网络节点，比如你连 wifi ，首先要经过路由器，路由器再经过一定的算法跳到下一个路由器（或其他网络设备），最终达到目的地之前，已经经过了 n  个网络设备，别人绝对可以拦截到你的信息的。


<br>
# 2. 对称加密 和 非对称加密

于是就出现了很多加密数据的办法。

### 对称加密

使用的密钥只有一个（本质就一字符串），发收信息双方都使用这个密钥对数据进行加密和解密。

### 非对称加密

公开密钥与私有密钥是一对，如果用公开密钥对数据进行加密，只有用对应的私有密钥才能解密；
如果用私有密钥对数据进行加密，那么只有用对应的公开密钥才能解密。
因为加密和解密使用的是两个不同的密钥，所以这种算法叫作非对称加密算法。

关于非对称加密，主要是下面几个点要搞清楚：

<br>
> 问题1：为什么公秘钥是两把不同的钥匙(字符串)，却能互相加解密？

答：本质其实是基于数学算法，用到了素数、互质数、指数运算、模运算等数学知识，要深入理解网上有很多教程。但是暂时只要知道两者基于数学算法，所以可以互相解密即可。

<br>
> 问题2：为什么公钥能给别人知道，私钥却不行？

答：

首先，秘钥能很快推算出公钥，但是公钥在短时间内很能推算出秘钥。所以即使公钥公开，秘钥也是相对安全的。所以一般才公开给别人公钥，而不是给私钥。

其次，公钥加密的信息只有私钥能解，别人有一样的公钥也解不了的。


<br>
# 3. 什么是 hash 函数？

知道了非对称加密，还需要理解一下 hash 函数，其实也是一个算法。

简单理解就是，我用 hash 函数对一段内容进行加密，得到一个唯一值，这个值很难被逆推出来原来的内容。由于这个值不可逆推，所以能用于验证某段内容的有效性。

最简单的例子就是登陆的时候的密码，存在数据库就是一个 hash 值。登录的时候对你输入的密码再进行一次 hash ，两个 hash 值进行对比去验证密码。

因为数据库存的不是你的真正密码，所以别人盗取了这个 hash 值也很难推出你的密码。（当然如果你的密码总是123456， 那存 hash 值也救不了你）


<br>
# 4. 数字签名

发送一段话，如何确保内容不被修改？

那就在发送的时候附上这段内容的 hash 值，接收方对这段话再进行 hash ，得到同样的 hash 值，说明内容没被修改。

 那如何保证再保障 hash 值也不会修改？就需要再对 hash 值进行一层加密（对称加密或非对称加密都行）了。这就是基本的数字签名的应用。
 
 <br>
# 5.  误区解释

好，上面看完了，估计对非对称加密还是有疑问，没错，我也有疑问，这尼玛不靠谱呀，假设我有一个秘钥，我的朋友和父母都有公钥，我和我的好友进行通信，这时候有两个不理解的地方：

> 公钥是公开的呀，所有人都能解密到内容呀?

> 拿到公钥一方就可以向秘钥发送信息呀，岂不是个个都能发?


其实这是因为我以为一套非对称加密就能解决所有问题，这是理解的误区。上面两个问题对应两种应用场景： RSA 签名方案 和 RSA 加密方案。

### RSA 签名方案 
 
第一种用法，只是用于签名，也就是证明私钥拥有者的内容不被串改伪造。

比如，我用哈希函数对 “今晚开黑” 进行 hash，得到一个 hash 值，用私钥加密这个 hash 值，附在信上发送给朋友。

我的开黑小伙伴用公钥解密得到 hash 值，同时也用同样的 hash 函数去对 “今晚开黑” 进行 hash，对比一下两个 hash 值。

中间只要有人劫持修改了内容比如 “今晚不开黑” ，就会导致 hash 值不同。或者公钥解密没起作用，说明 hash 值本身也被改了，小伙伴就知道事情不对劲了。

而所有有公钥的人都可以知道我今晚开黑，比如我的父母，这也是没办法的事情。

这种情况下：只能保证内容不被串改，不保证内容不被别人知道。比如你发的是一条公告新闻，允许每个人都看得到，但是别人伪造不了。

### RSA 加密方案

第二种用法，只单纯用于加密。这个加密是指，公钥加密的，只有私钥能解开，别人解不了。

比如小伙伴用公钥加密给我说，“好的，今晚开黑“，我的父母有公钥，也解不开小伙伴的内容。只有拥有私钥的我可以解开。

这种情况下：只能保证不被人解密出其中的内容，但是所有人都能给我发信息。如果要辨别出不同的人，那就要配合其他技术。


<br>
# 6.关于 jwt 

之所以提一下 jwt, 是因为我以前以为 jwt 就是非对称加密的主要应用，实际上并不是说 jwt 一定要用非对称加密。写在这里是为了提示自己修正一下认知。这一部分也可以跳过，不影响 https 的学习。

一般 jwt 的生成验证如下：

服务器验证账号密码后，生成一些 json 内容，包括一些基本的用户信息。然后再对内容进行某种加密（对称或者非对称都行），比如 HMAC，就是用一个秘钥，去对内容进行加密，然后附在用户信息后面, 最后将这些内容返回给浏览器。

浏览器后面每次发起请求都带上这段内容（token），服务端要认证，其实就是用秘钥对前面的用户信息再做一次 HMAC，然后判断是否和后面的内容一致，就可以知道前面的用户信息有没有被修改过，确保无误则承认该用户，用于后面其他操作。

> 注：HMAC 就是 hash 算法的一种升级应用。即能保障不可逆，又能保障多一个秘钥去混淆，不知道秘钥无法用普通的 hash 函数去伪装。


<br>
# 7. 证书

假设我们使用非对称加密去验证信息，那服务器就要下发一个公钥给客户端用于加密信息。

但是，对于请求方来说，它怎么能确定它所得到的公钥一定是从服务器那里发布的，而没有中间人篡改过呢？一旦这个公钥不可靠（比如黑客伪造的），那请求方发的内容就很有可能被黑客劫持并解密。

如果在线发送公钥，肯定要通过互联网传输，上面我们说过网络是不安全的，那如何保证约定的公钥不泄露? 再来一层非对称加密？那第二层非对称加密的公钥如何传输?这样就陷入死循环中了。

最好的办法就是把公钥直接送到你家里，给你手动输入到电脑。哈哈，可能吗？答案是可能，这就是 https 了。只不过你保存的不是某个网站的公钥，而是某个权威机构的公钥。

https 的权威机构（CA）和电脑操作系统或浏览器厂商合作，内置了自己的公钥（这就是我说的到你家给你输入），从而保障不可篡改。然后 CA 还会用自己的私钥加密一份内容（你的网址和你的服务器的公钥等信息），作为你网站的证书。

这就有了如下的 https 请求过程:

首先，客户端向服务器发出加密请求。 

服务器用自己的私钥加密信息后，连同本身从 CA 拿到的数字证书，一起发送给客户端。 

客户端（浏览器）的 ”证书管理器” 中有 ”受信任的根证书颁发机构” 列表。客户端会根据这张列表，查看解开数字证书的公钥是否在列表之内。

如果存在，就会用该权威机构的公钥解开证书，证书记录的内容就是网址和其他信息（比如域名），如果发现没有问题，就可以证明这张证书的确是 CA 机构发布的，浏览的这个网站也没有问题，那就取出证书里网站真正的公钥，开始加密传输。

一旦解不开或者发现信息对应不上，就不会再建立连接了。

可以看到关键点就在，这个 CA 机构的公钥是内置的，不是网络传输的，以此保证可靠。

<br>
# 8. 实际操作

上面说了那么多，可以看得出来，电脑保存了 CA 机构的公钥，CA 机构保存了你的公钥 和 CA 自己的私钥，服务器一端需要保存证书和服务器的私钥。

那么如何申请和配置就一目了然了：

### 1. 首先到腾讯那里申请一下证书

https://cloud.tencent.com/product/ssl

### 2. 下载证书

一般默认从 nginx 安装的根目录开始读证书和秘钥，所以下载解压后将其放在 nginx 的根目录就行

### 3. 修改 nginx 的配置


```
server {
        listen 443;
        server_name www.domain.com; #填写绑定证书的域名
        ssl on;
        ssl_certificate 1_www.domain.com_bundle.crt;
        ssl_certificate_key 2_www.domain.com.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
        ssl_prefer_server_ciphers on;
        location / {
            root   html; #站点目录
            index  index.html index.htm;
        }
    }


```

如果是 docker，需要映射 443 端口

```

docker run \
-p:443:443 \
-p:80:80 \
--name mynginx \
--net nginx-network \
-v "$PWD/html":/usr/share/nginx/html \
-v "$PWD/conf":/etc/nginx \
-v "$PWD/log":/var/log/nginx/ \
-v "/home/data/php/www":/usr/share/nginx/www \
-d hub.c.163.com/library/nginx  
```

详细教程可见：
https://cloud.tencent.com/document/product/400/4143

其实还有一个问题，下载的时候证书和私钥泄露了怎么办哈哈哈。如果真的极度强迫 + 担心，建议看看有没有那些私钥自己生成，公钥由自己上传的方案吧哈哈哈哈哈。

<br>
# 9. 其他

证书类型、多级证书机构等知识后面可以慢慢理解。

<br>
# 10. 总结
 1.不建议一开始就直接配置，而是先明白理论知识，后面就容易多了。
 
 2.理解了 https ，其实就能理解 guzzle 框架开启 验证 SSL 证书时为什么需要证书包，一些代理工具解析 https 时为什么需要修改本地证书等等问题。
 
 3.另外，https 只是保证网站数据传输安全（加密与不可篡改），但是如果这个网站一开始就是打算骗你钱的，那数据传输安全不安全也没意义了哈哈。

 4.修正之前的一个想法，就是以为服务器下发的信息也会被窃取，实际上是我理解错了。上面说到的 https 中的 非对称 加密，主要是第一次连接时用来约定双方的 对称 加密算法的，后面双方其实是用约定的 对称 加密算法来收发信息，所以对外都是不可见的。

 5.之前错误的想法：
~~从上面我们也知道，https 只是保证了浏览器发的信息只有服务器能解密，服务器下发的信息不可篡改而已，也就是说服务器如果下发用于 jwt 验证的 token 时 ，虽然不可改，但是有可能泄露，这点要注意。~~

<br>
##### 后记：果然某个阶段认为正确的事情，在下个阶段就会被颠覆。