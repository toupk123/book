# 第四 返回结果的HTTP状态

​	返回客户端结果的状态吗

1. 1xx 接受的请求正在处理
2. 2xx 请求正常处理完毕
3. 3xx 重定向状态码 需要进行附加操作完成请求
4. 4xx 服务器无法处理请求
5. 5xx  服务器错误状态码

## 2xx
200 正常处理，正常返回
204 请求成功处理，但是返回的响应报文中不含实体的主体部分。
206 因为客户端设置了范围请求，content-range设置后

## 3xx重定向
301 永久重定向
302 临时性重定向，还有可能会变更
303 和302相似，但是表示应该采用get方法获取资源
304 和重定向没有关系，只是表示客户端需要的资源已经找到，但客户端没有必要

## 4xx 客户端错误类型

400 表示请求报文中存在语法错误
401 需要进行认证
403 服务端拒绝了请求
404 没有找到资源

## 5xx服务器错误
500 服务器出现错误
503 服务器正在超负载 或者正在停机维护，无法处理相应

# 第五 与http协议的web服务器

## 用单台虚拟主机可以实现多个域名
物理层面的一台服务器，可以通过虚拟主机，达到多台服务器

## 通信数据转发程序

### 代理
代理是接受客户端的请求，然后转发给其他服务器，并将结果返回给客户端，需要添加Via首部字段信息标记经过的主机信息。

使用代理服务器的路由: 利用缓存技术，减少带宽的流量。组织内部针对特定网站的访问权限控制。

1. 缓存代理
代理转发响应时，缓存代理会预先将资源副本 保存在代理服务器上。
当请求相同资源时，就可以不从源服务器上获取，直接将之前缓存的资源作为相应返回。

2. 透明代理
不对响应做任何处理

### 网关
类似于代理，但是可以非http协议的清楚，用户端发起http请求，但是网关可以处理发起非http请求，给其他服务器

### 隧道
建立安全的隧道，使用ssl等加密手段进行通信

# 第六章 HTTP首部


# 第七章 确保安全的HTTPS

## HTTP的缺点
1. 通信使用的明文，内容可能会被窃听
2. 不验证通信方的身份，因此有可能遭遇伪装
3. 无法证明报文的完整性，所以有可能已遭串改


 ### 加密
当http与ssl（安全套接层）组合使用的时候，就被称为https

### http通讯时不会确认通讯方身份
即使毫无意义的请求也会照单全收，无法阻止海量请求下的Dos攻击。

#### 可以通过查看对方证书确定通讯方
使用ssl则可以确定，因为ssl不仅提供加密处理，还是用一种被称为证书的手段，可以用于确认方。
这些证书一般都是值得信赖的第三放颁发，客户端在通讯之前，会先确认证书，确定请求对象真实性


### 无法证明报文完整性，可能一遭到篡改
http协议没有办法保证服务器到客户端的文件是一样的，可能请求在中间被人拦截。
这里就需要ssl提供的认证和加密处理摘要。

## HTTP+加密+认证+完整性保护 = HTTPS
https并不是一个新的协议，只是披着ssl协议外衣的http协议。
原先http协议作为应用层是直接和tcp协议链接，但是现在http是直接和ssl协议连接。

## 相互交换秘钥的公开秘钥
SSL采用的叫做公开秘钥加密。
加密和解密都是用哪个的同一个秘钥的方式被称为共享密钥加密，对称密钥加密

使用公开秘钥加密方式就能解决共享密钥加密的困难。
1. 公开秘钥加密使用一对非对称的秘钥。一把叫做私有秘钥，另一把是公开秘钥。
使用公开秘钥传送东西，接受方再使用私有秘钥解密。

## https采用混合加密的形式
公开秘钥加密方式处理起来比较复杂，效率就会比较低。
所以https会先采用公开加密的方式，交换共享密钥后，接下来就会直接使用共享密钥的形式

### 证明公开秘钥正确性的证书
为了证明本身公开密钥的正确性，就会采用第三方认证机构，证明其安全性


## 第八章  认证用户身份
