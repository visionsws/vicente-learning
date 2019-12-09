# 一、前言 

这篇文章中我们来了解一下**JWT是何方神圣？**以及**JWT来实现分布式Session**。

# 二、JWT是什么

JWT一看就是简称，它的**全称JSON Web Token**，从字面上我们看出

> 1、数据是JSON格式
>
> 2、用于Web应用
>
> 3、是一个Token，也就是一个令牌方式

看看官方的说明，它定义了一种**紧凑且自包含**的方式，用于在各方之间以**JSON对象进行安全传输信息**。这些信息可以通过对称/非对称方式**进行签名，防止信息被串改**。

> **紧凑的含义：**就是**JWT比较小，数据量不大**，可以通过URL、POST参数或**Header请求头**方式进行传输。
>
> **自包含的含义：**jwt可以让用户自定义**JWT里面包含的用户信息**，如：姓名、昵称等（**不要放隐密的信息**）。从而**避免了多次查询数据库**。

# 三、JWT数据结构

- JWT由三个部分组成

> **1、Header**
>
> **2、Payload**
>
> **3、Signature**

- 三者组合在一起

```
Header.Payload.Signature
```

- 案例

![img](https://mmbiz.qpic.cn/mmbiz_jpg/UtWdDgynLdYoPticfIJPf0c6Y4xXOxNicnM7U0NFYF4jmpicicKr81rfa6PFlE3XrGsjxnPHSCETiaSFBCo1mRffVew/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



看上去是不是满乱，我们来依次看下里面的结构。

# 四、Header

这个是**JWT第一段数据，表示头部信息**，主要的作用是**描述JWT的元数据**，上面的案例就是：

```
{
 alg: "HS256",
 typ: "JWT"
}
```

> 1、alg属性表示**签名的算法，默认算法为HS256**，可以自行别的算法。
>
> 2、typ属性表示这个**令牌的类型**，JWT令牌就为JWT。

上面的JSON数据会通过**Base64算法**进行编码而成，看工具图

![img](https://mmbiz.qpic.cn/mmbiz_jpg/UtWdDgynLdYoPticfIJPf0c6Y4xXOxNicned3q5vUG65cK3Ir8DUYicTFnz2l0trCE0nohia63RNuAbtU46Wh42GPw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**五、Payload**

此为JWT第二段数据，用来存放实际需要传递的数据。JWT官方也规定了7个字段供选用

![img](https://mmbiz.qpic.cn/mmbiz_jpg/UtWdDgynLdYoPticfIJPf0c6Y4xXOxNicn1DaxblLqyUqmOibxXrGOuLibFH9nXfkkBf6JQ29oJaI8XiclZKzXndHgg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当然除了官方字段，我们**可以自定义字段**，以上面的案例，我们看下实际的数据

![img](https://mmbiz.qpic.cn/mmbiz_jpg/UtWdDgynLdYoPticfIJPf0c6Y4xXOxNicnTz1XRYZekZu5uJyoTVUmZwiaNUSBPQ0dgVTV7ZDRQbib2cnDodYVyRQw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 注意：这段也是用Base64算法，**JWT默认是不加密的**，任何人都可以获取，只要进行**Base64解码**就行了，所以**不要把隐密的信息放到JWT中**

# 六、Signature

此为JWT第三段数据，**主要作用是对前面两段的数据进行签名，防止数据篡改**。一般我们进行签名的时候会**有个密钥（secret）**，**只有服务器知道**，然后**利用Header中的签名算法**进行签名，公式如下：

```
HMACSHA256(
 base64UrlEncode(header) + "." + base64UrlEncode(payload),
 secret)
```

算出签名后，**把Header、Payload、Signature三个部分拼成一个字符串**，之间用（.）分隔，这样就可以把组合而成的字符串返回给用户了。

# 七、JWT的工作方式

在用户进行认证登录时，**登录成功后服务器会返回一个JWT给客户端**；那这个JWT就是用户的凭证，以后**到哪里去都要带上这个凭证token**。尤其访问受保护的资源的时候，通常把JWT**放在Authorization header中**。要用 Bearer schema，如header请求头中：

```
Authorization: Bearer <token>
```

![img](https://mmbiz.qpic.cn/mmbiz_jpg/UtWdDgynLdYoPticfIJPf0c6Y4xXOxNicndTAKxJC9yTiaVRCSQbGiaBYR0V27Bcfiamg9fLvfkPP4A6GRCYytv13GA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 八、基于JWT的身份认证

上面的JWT的工作方式，其实就是一个**完整的身份认证流程**，我们这里把这个讲的在通俗一点。

> 1、用户提供用户名和密码登录
>
> 2、服务器校验用户是否正确，如正确，**就返回token给客户端，此token可以包含用户信息**
>
> 3、客户端存储token，可以**保存在cookie或者local storage**
>
> 4、客户端以后请求时，都要**带上这个token，一般放在请求头中**
>
> 5、服务器判断是否存在token，并且解码后就可以**知道是哪个用户**
>
> 6、服务器这样就可以**返回该用户的相关信息**了

这个流程小伙伴们有没有发现，**用户信息是放在JWT中的**，**是存放在客户端（cookie，local storage）**中的，服务器**只需解码验证就行了，就可以知道获取到用户信息**。而我们之前的Session方式就不一样。

# 九、与Session-Cookie方式的区别

Session-Cookie方式的这里就不多作介绍了，之前文章已经介绍了。直接上图说明区别

![img](https://mmbiz.qpic.cn/mmbiz_jpg/UtWdDgynLdYoPticfIJPf0c6Y4xXOxNicnTBmyicm3mAfm3USnmMsYCRGxKbNDXr9ibCA8Pk4XLShcicE8NlSpAf6ibw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上图是Sesson服务器方式，我们发现Session用户信息是在**服务器端存储**的。

我们再来看看JWT方式

![img](https://mmbiz.qpic.cn/mmbiz_jpg/UtWdDgynLdYoPticfIJPf0c6Y4xXOxNicnaEzokMxbxbLKYq8XYjCGFLqic2srnN9rywQw1jMR6qgASiaq75DI7HJA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上面的token即用户信息是存储在客户端的，服务器端只要解码即可。

# 十、JWT方式认证的好处

> 1、因为token存储在客户端，服务器只负责解码。这样**不需要占用服务器端资源**。
>
> 2、服务器端可以**无限扩展**，负载均衡器可以将**用户传递到任何服务器**，服务器都能知道用户信息，因为jwt里面包含了。
>
> 3、**数据安全，因为有签名**，防止了篡改，但信息还是透明的，不要放敏感信息。
>
> 4、放入请求头提交，很好的**防止了csrf攻击**，

说了这么多的好处，那是不是**JWT就非常适合替换掉Session方式**呢？

# 十一、JWT方式的坏处

**1、token失效问题**

JWT方式最大的坏处就是**无法主动让token失效**，小伙伴们会说token不是有过期时间吗？是的，token本身是有过期时间，但**token一旦发出，服务器就无法收回**。

> 如：一个jwt的token的失效时间是3天，但我们**发现这个token有异常**，有可能被人登录，那真实的用户可以修改密码。但是即使修改了密码，**那个异常的token还是合法的**，因为3天的失效时间未到，我们**服务器是没法主动让异常token失效**。

**2、数据延时，不一致问题**

还有个问题就是因为jwt中**包含了用户的部分信息**，如果这些**部分信息修改了**，服务器获取的还是**以前的jwt中的用户信息**，导致数据不一致。

# 十二、总结

小伙伴们**怎么去选择Session的方式**，是用**传统的Sesion-Cookie服务器**方式，还是用**JWT方式**，具体集合业务看。不过老顾这里**推荐还是用传统的方式**，因为以后的业务很有可能会用到用户Session。好了，谢谢！！！