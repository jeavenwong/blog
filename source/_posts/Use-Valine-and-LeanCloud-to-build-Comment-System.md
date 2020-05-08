---
title: Use Valine and LeanCloud to build Comment System
date: 2020-03-28 14:57:44
tags: hexo
---

原始的hexo的模板landscape是没有评论系统的，我们可以自己定制一个评论系统。

<!--more-->

可以使用 github 或者 gitee（码云）建一个第三方应用来做博客系统，但是样式不够简洁，而且评论者需要登录 github 或者 gitee账号，这里我们选用一个免费的开源博客系统 Valine，并且 Valine 默认使用云存储平台 LeanCloud 来做评论的存储，LeanCloud的开发版的存储是免费的。

Valine 的初始配置详细见官网：

[Valine 官方手册]: https://valine.js.org/quickstart.html

注册 LeanCloud 并获得了 AppID 和 AppKey 之后就可以对 landscape 进行配置了。

编辑 `/themes/landscape/layout/_partial/`目录下的`article.ejs`，将原本配置gitment时添加在最后的那段代码删掉，添加： 

```js
<% if (!index){ %>
  <% if (post.comments){ %>
    <div id="vcomments"></div>
    <script src="//cdn1.lncld.net/static/js/3.0.4/av-min.js"></script>
    <script src='//unpkg.com/valine/dist/Valine.min.js'></script>
    <script>
        new Valine({
            el: '#vcomments',
            appId: '你的appid',
            appKey: '你的appkey',
            notify:true, 
            verify:true, 
            visitor:true,
            avatar:'mm', 
            placeholder: '嘻嘻嘻' 
        })
    </script>
  <% } else { %>
    <div class="vcomments"></div>
  <% } %>
<% } %>
```

之后，用 hexo 重新编译然后上传到 gitee pages 即可。

运行效果图如下：

![效果图](valine_demo.JPG)



参考链接：[将hexo的评论系统由gitment改为Valine](https://www.cnblogs.com/zmj97/p/10180732.html)

