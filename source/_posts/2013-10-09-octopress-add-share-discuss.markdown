---
layout: post
title: "octopress添加社会化分享和添加多说评论"
date: 2013-10-09 16:14
comments: true
categories: octopress
---

###社会化分享
社会化分享主要采用的是bshare

在\_config.yml中增加bshare:true,在"source/\_includes/post"下的share.html中添加如下代码

```js
<div class="bshare-custom"><a title="分享到QQ空间" class="bshare-qzone"></a><a title="分享到新浪微博" class="bshare-sinaminiblog"></a><a title="分享到人人网" class="bshare-renren"></a><a title="分享到腾讯微博" class="bshare-qqmb"></a><a title="分享到网易微博" class="bshare-neteasemb"></a><a title="更多平台" class="bshare-more bshare-more-icon more-style-addthis"></a><span class="BSHARE_COUNT bshare-share-count">0</span></div><script type="text/javascript" charset="utf-8" src="http://static.bshare.cn/b/buttonLite.js#style=-1&amp;uuid=&amp;pophcol=2&amp;lang=zh"></script><script type="text/javascript" charset="utf-8" src="http://static.bshare.cn/b/bshareC0.js"></script>
```
这样就完成了一键分享的功能。

###社会化评论
octopress默认采用第三方评论系统disqus，但是国内用户普遍都不使用，可以采用duoshuo。
首先在 source/post/ 下创建duoshuo.html
然后登录duoshuo的网站，进去获得代码后复制到刚刚创建好的html中。
在source/\_layouts/post.html中，将对应的disqus代码改为：
```js
{% if site.duoshuo_name and page.comments == true %}
  <section id="comment">
    <h1>发表评论</h1>
    {% include post/duoshuo.html %}
  </section>
{% endif %}
```
在source/\_config/yml中，添加：
```sh
duoshuo_name: 你在多说创建的站点名称
```

不出意外，分享和评论就添加完成了，最后不要忘记生成就可以了。


