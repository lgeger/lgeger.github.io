---
title: 自己动手做一个Server酱-cloudflare workers实现通过微信公众号发送消息
date: 2023-08-09 18:01:17 +0800
categories: [工具]
tags: [工具]     # TAG names should always be lowercase
---


## 起因
不知道大家有没有用过server酱，一开始发现这个工具的时候，我觉得也太好用了，好多场景都可以用起来，我也一直在用，不过后来也出现了收费还有限制条数这些情况，并且作者其实提供了开源版的实现，所以我一直想自己搞一个自己用的。要求就是我给某个地址发消息，消息可以通过微信收到就可以。
## 原理
server酱的原理其实很简单，其实就是将服务号/测试号的接口进行了封装，让我们可以只关心要发送的内容，不用处理鉴权和其他一些参数要求。
## 准备
一个服务号，或者申请一个测试号：[申请地址](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)
另外，我会将代码部署在cloudflare woker上，所以还需要一个cloudflare账号，可以点击这里申请：[地址在这里](https://www.cloudflare.com/)
## 开搞
完成这个功能，我们需要以下几个东西
* appID
* appsecret
* userId
* templateId

前两个可以在服务号里找到，如果是心申请的测试号的话，页面上直接就可以看到：
![在这里插入图片描述](https://img-blog.csdnimg.cn/1967139606e9428abf6e9fe995c2ccb7.png)
然后在下面的模板消息里，新建一个模板消息：
![在这里插入图片描述](https://img-blog.csdnimg.cn/32f6fda5b55148e58d029c2601d974a5.png)
注意，模板内容的部分，要用占位符：

```java
标题：{{title.DATA}}
内容：{{content.DATA}} 
```
我自己测试，如果不写标题和内容这几个字，只剩下占位符的话，收到的消息看不到内容。然后可以自己添加自己需要的字段，自己定就可以。
## 参考资料
参考了这几个博客，有需要的也可以看看：
[自己动手做一个Server酱·TurboMini版](https://mabbs.github.io/2021/02/02/serverchan.html)

[Cloudflare Workers 初探——以 G2WW 作为例子转发 Grafana 报警到企业微信](https://nova.moe/first-touch-with-cf-workers/)

[CloudFlare Workers 流量转发代码 支持非标准端口](https://www.blueskyxn.com/202102/4163.html)
## 实现
### 在cloudflare worker上新建一个worker

从这里开始
![在这里插入图片描述](https://img-blog.csdnimg.cn/7f03633fc01643008113583cfdc4596b.png)
建一个worker

![在这里插入图片描述](https://img-blog.csdnimg.cn/56ab37629c1c4545a5839f80b5a5869a.png)
Deploy
![在这里插入图片描述](https://img-blog.csdnimg.cn/5e4add6154194db5939aa1f7097efebb.png)

这样我们就得到了一个worker，并且得到了可以访问的域名，不过cloudflare worker提供的域名已经被ban了，可以使用自己的域名。

### 自己改改代码粘贴进去
放到了github上，地址是：[cloudflare-wechat-message](https://github.com/lgeger/cloudflare-wechat-message)

### 配置缓存
因为代码里有一段将access_token用到了缓存，存储到了kv里，所以这里写一下kv如何配置，写一下纯页面配置：
首先新建一个 kv
![在这里插入图片描述](https://img-blog.csdnimg.cn/f81ac3b521ae4ee5a72c68d6246c9dfa.png)
然后回到我们的worker，设置→变量→KV 命名空间绑定→编辑变量
![在这里插入图片描述](https://img-blog.csdnimg.cn/58776f1cdab44019819a83f2aced1ea3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/db55a85447ad4d7e84f346047dd3b446.png)

注意注意，这段代码里虽然用了缓存，但是我完全没做异常处理，token过期的情况我没处理，回头再改
### 修改成自己的域名
点进新建的worker页面，然后选择tigger→Custom Domains，添加自己的域名。

![在这里插入图片描述](https://img-blog.csdnimg.cn/af4745b6b7f449cb84558fc94e17a354.png)

搞定！看看效果：
请求一下自己的链接，比如：

```java
https://xxxx.workers.dev/?title=aaaaaa&content=hhhhhhhh
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/3f36d7365cf3467193a723beddcd968d.png)
### 结语
实现很粗糙，我的js实在是太抠脚了，大家有好的实现请快快评论告诉我吧，感谢