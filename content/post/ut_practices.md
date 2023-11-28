---
title: "Python UnitTest 实践记录"
description: "UnitTest 是软件开发中非常重要的部分"
keywords: "UT UnitTest 单测"

date: 2023-11-29T00:10:46+08:00
lastmod: 2023-11-29T00:10:46+08:00

categories:
  - 技术
tags:
  - 单元测试
  - UnitTest

# 原文作者
# Post's origin author name
#author:
# 原文链接
# Post's origin link URL
#link:
# 图片链接，用在open graph和twitter卡片上
# Image source link that will use in open graph and twitter card
#imgs:
# 在首页展开内容
# Expand content on the home page
#expand: true
# 外部链接地址，访问时直接跳转
# It's means that will redirecting to external links
#extlink:
# 在当前页面关闭评论功能
# Disabled comment plugins in this post
#comment:
#  enable: false
# 关闭文章目录功能
# Disable table of content
#toc: false
# 绝对访问路径
# Absolute link for visit
#url: "ut_practices.html"
# 开启文章置顶，数字越小越靠前
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# 开启数学公式渲染，可选值： mathjax, katex
# Support Math Formulas render, options: mathjax, katex
#math: mathjax
# 开启各种图渲染，如流程图、时序图、类图等
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---

理论上说，单元测试在软件开发中非常重要，然后国内互联网环境下，要想安静写好单测实在太困难。
在大厂里，测试和研发部门基于各自立场往往围绕单测覆盖率打架，年初也许能协商好一个指标，但年中项目压力一上来，单测指标往往迅速被放弃；
而在创业公司里，代码能 5 分钟写完跑起来就是胜利，写单测要考虑代码抽象、测试 case，太慢不太符合调性。

不过，存在即合理。经历过一些血泪教训，才知道多写两个单测，也许可以避免很多低级又致命的错误。回忆过往，笔者至少有 2 次线上影响面较大的 case，
都是很低级的 BUG，但因为着急上线单测未覆盖而未被检出。这两个 BUG 大致是

1. 一个 pointer vector 新增修改逻辑，在 size 改变的情况下有 bug 会导致越界
2. 一个 while 循环生成 unique id 的逻辑，没有自增 id 导致生成的 id 总是相同，最终数据被相互覆盖

写出来都是很简单的问题，但写代码时自缘身在此山中，又或者疲劳赶工，往往难以发觉；而单纯端到端跑离线 case，在程序规模较大时基本难以覆盖所有条件。
而最终代码上线，在用户千奇百怪的输入中，任何未被测试覆盖的条件都可能被触发。上述低级翻车的尴尬惨剧就轻易发生了。

所以，如果你要开始着手写一个规模大、预期鲁棒的系统，单测必不可少。
本文将近期学习、实践的 Python UT 经验记录下来，期望给有志于不断精进代码能力的同行一点微小的帮助。

<!--more-->
