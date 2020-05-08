---
title: 如何使用hexo发布文章
date: 2020-03-28 08:44:52
tags: 
- hexo  
- 发布博客
---

######   前言

​	个人博客的好处多多，不仅可以记录下自己的学习过程中的踩坑经历，也可以写下自己的所思所想。

<!--more-->

搭建博客的方式其实很多，比如，本人是主攻 Java Web后端方向，可以采用 Spring Boot + MyBatis + Mysql + Docker 来搭建博客后台，用 jQuery/Bootstrap/Vue.js + node.js 来做博客的前台，这样的技术栈来实现一个单机版的个人博客的难度应该不会很大，主要就是付出时间和精力来coding 和 debug。但是考虑到要配置部署博客，购买服务器、域名，甚至购买HTTPS的证书等，这些都是需要投入钱和时间来做，出了问题还得定位问题来 debug，还得做好博客备份。所以个人感觉要维护一个高质量的个人博客要花费很多精力和金钱，这性价比并不高。

​	国内有很多优秀的博客平台，博客园 / CSDN  等，CSDN 不仅广告多，而且这几年质量下滑的严重，博客园允许个人定制页面的样式，但是定制受限，往往也不够简洁。出于想要做一个没有侧边栏广告且简洁明了的个人博客的初衷，选定了 github pages 来托管自己的静态博客，可是国内访问 github 的速度受限，为了体验更好于是选择了相似的国产的 gitee pages 来托管博客，Hexo 是一个比较成熟的博客框架，gitee pages 也支持，可以实现静态博客的 SSR (Server Side Render)。

​	本文就记录下日常使用hexo的方式。



首先在本地安装node.js，之后安装包管理器npm，这里省略具体过程

1. 使用npm下载hexo

   ```bash
   npm install hexo
   ```

2. 用hexo生成博客根目录的source/_post目录下新的markdown文件

   ```bash
   hexo new
   ```

    或者

   ```bash
   hexo n
   ```

3. 编辑生成的 markdown 文件的内容即可，这里推荐 Typora 软件

   在 blog 根目录下的 _config.yml 把 post_asset_foler:false 修改为 true 后，上一步会生成一个同名文件夹，在里面可以放置图片素材

   因为下载了如下相关插件，所以引用直接是图片名称即可，如 test.jpg 而不是 xxx/test.jpg

   ```bash
   npm install https://github.com/7ym0n/hexo-asset-image --save
   ```

   记得在发布文章的时候如果想要在主页显示折叠的文章，记得在 md 文件里添加下面的标签

   hex o提供的   `<!-- more --> `  

   *注意：不能写在除了标题外的正文第一行，否则无效*

   ![折叠文章操作图](more_essay_test.JPG)

   ![折叠文章效果图](more_essay_show.JPG)

4. 清空之前生成的 css 等样式文件

   ```bash
   hexo clean
   ```

5. 生产新的样式文件

   ```bash
   hexo g
   ```

   或者

   ```bash
   hexo generate
   ```

6. 部署文章同步到 gitee pages 仓库的 {username} 仓库上即可

   ```bash
   hexo deploy
   ```

   或者

   ```bash
   hexo d
   ```

7. 如果想要在本地预览的话，可以执行下面的命令

   ```bash
   hexo s
   ```

    或者

   ```bash
   hexo server
   ```

   我在 gitee 新建了一个 [repo](https://gitee.com/jeavenwong/blog) 来同步自己本地的 blog 文件，所以得记得及时 push.

   如下图所示

   ![hexo_server_demo](hexo_server_demo.JPG)

   之后在浏览器输入localhost:4000即可在本地预览博客
   
   ![chrome_access_demo](chrome_access_demo.JPG)

这里吐槽一下，虽然在国内访问 gitee 比 github 快，但是 gitee 个人版每次从本地同步博客之后，都需要在 [gitee page 应用]( https://gitee.com/jeavenwong/jeavenwong ) 仓库里手动点击 pages 服务来进行更新。