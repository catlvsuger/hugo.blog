+++
title = "开启 Hexo 博客之旅"
date = "2018-10-30T08:00:00+08:00"
categories = "Hexo"
tags = ["Hexo"]
description = ""
#image = null  # To use: uncomment and replace null with value
#link = null  # To use: uncomment and replace null with value
slug = "Open the Hexo blog tour"
+++

<p class="description"></p>



## Hexo + github pages 搭建博客 ，速度快的飞起，直接按网上的步骤开始

**参考如下，可以直接按第二个去弄，主题是 next**
>- 使用 hexo+github pages 搭建博客：<http://laijianfeng.org/2018/05/使用hexo-github-pages搭建博客/>

>- **打造个性化博客（极致详细）**：<https://reuixiy.github.io/technology/computer/computer-aided-art/2017/06/09/hexo-next-optimization.html>
>- 修改文章内链接样式｜hexo： <https://blog.csdn.net/qw8880000/article/details/80235648>
- hexo 的 next 主题个性化教程：打造炫酷网站：<https://blog.csdn.net/qq_33699981/article/details/72716951?utm_source=blogxgwz1>
- 在 NexT 中使用 Valine 评论系统：<https://reuixiy.github.io/technology/computer/computer-aided-art/2018/07/15/use-valine-in-theme-next.html>
<!-- more -->

**安装没什么问题，个性化优化过程有点繁琐** 

## 遇到问题有以下几点：
>- 图片本地可以，上传到 github 除了主页，其他的跳转后显示有问题，图片路径修改为 github 的存储路径就好了，也可以用七牛云存储图片，但给的测试域名只有一个月有效期，长期的必须要备案的域名才行，后续在处理图片存储 (目前处理方式是通过简书编辑，简书会自动上传图片，然后使用简书的图片地址)

>- 页面 CSS 样式调整，比较费时，有前端的朋友找他们比较快速
- 用谷歌浏览器调试，内存飞速溢出，网站直接打不开，换火狐就好了（试了下 **reuixiy** 的博客，同样的问题，那就不是我安装的问题，后续有时间再看）
- 其他的问题在 reuixiy 的博客里都可以找到答案
- YAMLException: end of the stream or a document separator is expected at line 2 好一番检查才发现第一行 --- 变成 -- 两个了 **（特别注意符号，英文: 和空格）**
- 置顶无效，查看~/blog/node_modules/hexo-generator-index-pin-top/lib/generator.js 发现改成strict 的又还原回top
- 网易云音乐很多歌无法生成外链，这是个问题
- 底部有空白： 将 css 中 .footer  的 bottom: 0;去掉， 内联没找到位置，代替做法：在../Blog/themes/next/source/css/_custom/custom.styl 添加 .footer { bottom: initial;} 

---

话不多说，接下来就是开始我的博客之旅~~~ 

<hr />

