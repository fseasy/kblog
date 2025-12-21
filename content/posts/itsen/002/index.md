---
title: "#002 读文档学英语 | Get started with Quill"
date: 2025-12-21T20:07:59+08:00
slug: "quill-doc"
draft: false
image: 
math: false
license: CC BY-SA 4.0
comments: true
hidden: false

tags:
  - "Quill"
categories:
  - "IT's English Channel"
---

<!-- https://quilljs.com/docs/quickstart  -->


Hello，各位读者和听众朋友们，欢迎来到 FS IT's English Channel，这是我们的第二期内容。
前面 2 期内容都有点长，制作周期比较久，and the posts got practically no traffic, as expected :) 我这心里咚咚打起了退堂鼓。但是说好的更 6 个月看看，咱肯定不能现在就放弃了；不过肯定要控制下工时了。

今天我在调研 flutter 富文本编辑器时，毫无意外地发现了 Quill 这个牛皮的库。去看了下它的入门文档，写得是真的好啊。这就赶紧拿来做素材吧。


## Prerequisite 

开始看 QuickStart 前，先来看 2 个前置单词吧：

<section class="kp">
  <span class="dot"></span>

WYSIWYG
</section>

stands for "What You See is What You Get", pronounced "wiz-ee-wig". 开发环境下一般就指代富文本编辑器。看下面的使用例句：

<section class="callout">
  <span class="hdot"></span> WYSIWYG 例句

- WYSIWYG describes software where what you see while editing is what you get in the final result.
- What is the best WYSIWYG currently in Flutter right now?

</section>

---

<section class="kp">
  <span class="dot"></span>

quill
</section>

羽毛笔，Quill 网站的 ico 就是它 ![quill-ico](quill.ico). 看几个例句：

<section class="callout">
  <span class="hdot"></span> quill 例句

- A quill is a pen made from a large bird’s feather, used in the past for writing with ink.
- The manuscript was written in ink and quill.
- Quill is widely used for building rich text editors in web applications.
</section>

## Get Started With Quill

好啦，我们来看看 QuickStart 的内容吧，只有 3 个自然段。

<section class="st">
  <section class="src-tag">“ Source Text</section>

The best way to get started is to try a simple example. Quill is initialized with a DOM element to contain the editor. The contents of that element will become the initial contents of Quill.

</section>

非常简洁流畅的一段，直接引出 simle example, 并对 initialization 做出说明。下面是它对应的中文翻译，试试你能不能基于中文翻译，写出类似的英语表达呢？

> 中文翻译 (ChatGPT)  
> 最好的入门方式是从一个简单的示例开始。Quill 需要用一个 DOM 元素进行初始化，用来承载编辑器；该元素中的内容会作为 Quill 的初始内容。

我大概会这么翻译（已经经过 ChatGPT 修正语法）：

> Translation of FS  
> The best way to get started is from a simple example. Quill needs a DOM element to initialize, and the editor will be rendered at that DOM element. The contents of the DOM element will be the initial contents of Quill.

哇，是不是又臭又长。我让 ChatGPT 给优化一下，结果如下：

> Optimized Translation (natural)   
> The best way to get started is with a simple example. Quill requires a DOM element to initialize, and the editor will be rendered inside that element. The contents of the DOM element will serve as the initial content of Quill.

修改点如下：

- "from a simple example" → "with a simple example"，with 更准确
- needs → requires，requires 更正式
- "at that DOM element" → "inside that element", 去掉冗余的 DOM, inside 更准确
- "will be the initial contents" → "will serve as the initial", 这个改动比较神

还有一个更口语的版本：

> Optimized Translation (informal)   
> The easiest way to get started is with a simple example. Quill needs a DOM element to work, and the editor will show up inside that element. Whatever’s already in the DOM element will become the initial content in Quill.

这个读起来感觉也不错。

但是啊但是，相比原文档都长了很多——最关键的点就是 “Quill 需要用一个 DOM 元素进行初始化，用来承载编辑器”，在原文里就一句话： Quill is initialized with a DOM element to contain the editor. 如果把中文翻译成 “Quill 是从一个用来承载编辑器的 DOM 元素初始化的” 应该就更容易实现精确地反向翻译了；只不过这个中文句子有点拗口。

通过 inverse translation 并对比区别，是不是感觉更能体会到中英表达的差异，更加深了对 Native English 的感觉呢 :)

---

下面一段只有一句话

<section class="st">
  <section class="src-tag">“ Source Text</section>
And that's all there is to it!
</section>

能够猜到大概什么意思，但细究起来是不是感觉有点理解不了？我以前和公司财务对话就这个感觉——都是很简单的单词，但合起来，不知道 the exact meaning.

询问 ChatGPT, 这是一句非常口语化的表达，意思是 “事情就这么简单” / “就这些别别的了”，常用来教程或解释之后。

可以拆开下面的部分方便理解：

- that’s all：就这些
- there is：存在的部分
- to it：关于这件事

当然，我们应该去把整句记住，争取脱口而出。

看下面的例句：

<section class="callout">
  <span class="hdot"></span> And that's all there is to it 例句

- To make this recipe, just mix flour, sugar, and eggs. Bake for 20 minutes, and that’s all there is to it.
  做这个食谱，只需混合面粉、糖和鸡蛋，烤 20 分钟，就这么简单。

- Setting up the app is easy — download it, install it, and that’s all there is to it.
</section>

Bonus, 这句话的常用变体：

<section class="callout">
  <span class="hdot"></span> That's all there is to it 变体
- That's really all there is to it. (加 really)
- That's pretty much all there is to it. （pretty much, 口语高频词 = almost/nearly/basically）
- that's it （很口语、略直接）
</section>

---

最后一段：

<section class="st">
  <section class="src-tag">“ Source Text</section>

The real magic of Quill comes in its flexibility and extensibility. You can get an idea of what is possible by playing around with the demos throughout this site or head straight to the Interactive Playground. For an in-depth walkthrough, take a look at How to Customize Quill.

</section>

这段话里同样使用了大量地道口语，我们来一网打尽。

<section class="kp">
  <span class="dot"></span>

comes in
</section>

这里指“体现在xxx”。类似的词是 lies in, 但更正式。它还有其他的一些常见含义，我们以后遇到了再说。


<section class="callout">
  <span class="hdot"></span> comes in 例句
- The difficulty of the task comes in the details.
- The beauty of the design comes in its simplicity.
- The advantage comes in how easy it is to use.
</section>


<section class="kp">
  <span class="dot"></span>

get an idea of …
</section>

了解 / 大致知道。比 understand 更自然随意。看下面例句：

<section class="callout">
  <span class="hdot"></span> get an idea of … 例句
- This tutorial will help you get an idea of how the software works.
- By looking at the diagram, you can get an idea of the system architecture.
- He explained the process so I could get an idea of what was happening.
</section>


<section class="kp">
  <span class="dot"></span>

play around with …
</section>

玩xx / 随便试xx。非常口语化，比 experiment with 更轻松。看下面例句：

<section class="callout">
  <span class="hdot"></span> play around with … 例句
- You can play around with the settings to see what works best.
- I spent the afternoon playing around with the new API.
- I’m just playing around with the new library to see what it can do.
</section>


<section class="kp">
  <span class="dot"></span>

head straight to …
</section>

直接去 / 直奔。非常地道口语，比 go directly to 更生动。

<section class="callout">
  <span class="hdot"></span> head straight to … 例句
- Head straight to the "Settings" tab to configure the options.
- Users can head straight to the login page from the homepage.
- Click the icon to head straight to the advanced options.
</section>

<section class="kp">
  <span class="dot"></span>

an in-depth walkthrough
</section>

深入地**逐步**学习。walkthrough 指 step-by-step guide or demonstration.
类似表达的有 "a detailed guide", "a step-by-step guide".

<section class="callout">
  <span class="hdot"></span> an in-depth walkthrough 例句

- The tutorial provides an in-depth walkthrough of the installation process.
- We prepared an in-depth walkthrough of the new feature for developers.
- This video offers an in-depth walkthrough of the software interface.
</section>

<section class="kp">
  <span class="dot"></span>

take a look at …
</section>

看一看。非常常用的口语，比 read/check 等都口语。

<section class="callout">
  <span class="hdot"></span> take a look at 例句

- Take a look at the new feature in the dashboard.
- I'll take a look at the code to see what's wrong.
- Please take a look at the documentation before you start.
</section>

## End

好了，Quick Start 部分就看完了，我们这期的内容也结束啦。短短的一篇文档，蕴含着这么多的地道表达，一定要多多练习，内化为自己的东西哈，与君共勉了。

喜欢我们的内容，就点赞收藏吧，咱们下期再见。