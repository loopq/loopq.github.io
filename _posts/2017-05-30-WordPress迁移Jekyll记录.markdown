---
layout:     post
title:      "WordPress 迁移 Jekyll"
subtitle:   " \"简洁的博客带给人好心情\""
date:       2017-05-30 11:00:00
author:     "Hux"
header-img: "img/post-bg-2017-05-31.jpg"
catalog: true
tags:
    - Python
---

> “Yeah It's on. ”

## Started ##


<p>端午之后第一天，来记录一下前几天完成的博客迁移。</p>
<p>从WordPress迁移到Jekyll的原因主要有以下两点：</p>
1. WordPress的主题风格原因，WordPress的风格大多一致，且不够简洁
2. WordPress对MarkDown格式支持不是太好，就因为这个，我弄了好几套方案，显示都不太满意
3. WordPress需要放在服务器上，国内的服务器比如阿里、腾讯，国外的VPS厂商都需要月租，这也是一笔开销

好，下面来开始博客的搭建过程

##  Why Jekyll？  ##

前段时间逛简书的时候看到[MrFu](http://www.jianshu.com/u/c1b9d1faa7b5)的Glide系列质量不错且都比较完整，顺藤摸瓜摸到了他的博客，瞬间被他的博客Geek风吸引住，又顺藤摸瓜找到了他的主题的来源[主题](https://mrfu.me/life/2015/07/19/the_new_blog_theme/)，觉得我自己以前的WordPress博客真是笨重又蠢的慌，经不住吸引，我决定将自己的博客迁移过来，这就是我选择Jekyll的理由。

## How To Started？ ##

<p>配置Jekyll的环境是最麻烦也是最要命的。我是参考简书上的文章，链接在下面，参照一步步来就可以。</p>

[http://www.jianshu.com/p/5ad3e9b98d3e](http://www.jianshu.com/p/5ad3e9b98d3e)

[http://jekyllcn.com/docs/usage/](http://jekyllcn.com/docs/usage/)

依照上面两个链接，尤其是第一个链接一步一步来，可以搭建完成，完成上面的步骤才能进行下面

## why not github pages? ##
<p>这里我不明白的是网上似乎都是使用github pages作为容器，但是github pages的两大弊端就是</p>

1. 无法被百度爬虫索引到
2. github经常发疯，并且国内访问速度局限，最好的ping可能也得100+,我搭建博客使用的是[码市](https://coding.net)，免费+国内访问速度快+可以被百度索引，why not?

## 更换主题 ##

上面已经说了，我们要做成[这个大帅哥的主题](http://huangxuan.me/)，他也将他的改善后的主题开源放在了[github](https://github.com/Huxpro/huxpro.github.io)，使用git将他的主题clone下来，基本上不用配置什么，直接使用jekyll server就可以启动起来，浏览器输入localhost:4000就可以看到主题的首页，现在这就是你的主题了。嫌样式不对你的口味？可以自己修改。

## 发布至Coding & 自定义 域名 ##

1. 发布至Coding照常使用Git
2. 自定义域名
	1. 首先，你得打开Pages 服务![](http://i.imgur.com/aB1lNiZ.png)
	2. 指向你已购买的域名![](http://i.imgur.com/o77wibB.png)
	3. 切记在你的域名管理面板中添加 CNAME 记录指向到 pages.coding.me
	

<h2><P align="center">Ending</p></h2>









