---
title: "{{ replace .Name "-" " " | title }}"
description: "{{ .Name }}"
keywords: "{{replace .Name "-" ","}}"

date: {{ .Date }}
lastmod: {{ .Date }}

categories:
  -
tags:
  -
  -

# 原文作者
#author:
# 原文链接
#link:
# 图片链接，用在open graph和twitter卡片上
#imgs:
# 在首页展开内容
#expand: true
# 外部链接地址，访问时直接跳转
#extlink:
# 在当前页面关闭评论功能
#comment:
#  enable: false
# 关闭文章目录功能
#toc: false
# 绝对访问路径
#url: "{{ lower .Name }}.html"
# 开启文章置顶，数字越小越靠前
#weight: 1
# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax
# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---
