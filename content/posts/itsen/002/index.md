---
title: "Issue 002 读文档学英语 | Get started with Quill"
date: 2025-12-21T20:07:59+08:00
slug: "quill-doc"
draft: false
image: quill-logo.svg
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

前面两期内容都有点长，制作周期比较久，and the posts got practically no traffic, as expected :) 我这心里咚咚打起了退堂鼓——不过说好的更 6 个月看看，咱肯定不能现在就放弃了。投入肯定要控制下。简单回顾一下，录音、剪辑花了比较多时间，性价比太低，后续每周就只录 1 期的音频，其余期只做 TTS 音频。这样算是在保留核心功能的条件下降低了成本。就再跑下看看。

今天我在调研 flutter 富文本编辑器时，毫无意外地发现了 Quill 这个牛皮的库。去看了下它的入门文档，写得是真的好啊。这就赶紧拿来做素材吧。


## Prerequisite 

开始看 QuickStart 前，先来看 2 个前置单词吧：

<section class="kp">
  <span class="dot"></span>
WYSIWYG
</section>

stands for "What You See is What You Get", pronounced "wiz-ee-wig". 像 MS Word 这种就是 WYSIWYG，开发环境下一般就指代富文本编辑器。

<section class="callout">
  <span class="hdot"></span> WYSIWYG 例句

- WYSIWYG means "What You See Is What You Get". It describes software in which the content looks the same while you edit it as it will when it is finished or published.
- Microsoft Word and Google Docs are classic examples of WYSIWYG editors where your formatted text on screen appears almost identically when printed or shared.
- What is the best WYSIWYG currently in Flutter right now?
</section>

---

<section class="kp">
  <span class="dot"></span>
quill
</section>

羽毛笔，Quill 网站的 ico 就是它 ![quill-ico](quill-ico.png). 看几个例句：


<section class="callout">
  <span class="hdot"></span> quill 例句

- A quill is a pen made from a large bird's feather, used in the past for writing with ink.
- The manuscript was written in ink and quill.
- Quill is widely used for building rich text editors in web applications.
</section>

## Get Started With Quill

好啦，我们来看看 QuickStart 的内容吧，只有 3 个自然段。

第一段如下：

<section class="st">
  <section class="src-tag">“ Source Text</section>

The best way to get started is to try a simple example. Quill is initialized with a DOM element to contain the editor. The contents of that element will become the initial contents of Quill.

</section>

非常简洁流畅的一段，直接引出 simple example, 并对 initialization 做出说明。下面是它对应的中文翻译，试试你能不能基于中文翻译，写出类似的英语表达呢？

> 中文翻译 (ChatGPT)  
> 最好的入门方式是从一个简单的示例开始。Quill 需要用一个 DOM 元素进行初始化，用来承载编辑器；该元素中的内容会作为 Quill 的初始内容。

我大概会这么翻译（已经经过 ChatGPT 修正语法）：

> Translation of FS  
> The best way to get started is from a simple example. Quill needs a DOM element to initialize, and the editor will be rendered at that DOM element. The contents of the DOM element will be the initial contents of Quill.

哇，是不是又臭又长。我让 ChatGPT 给优化一下表达，结果如下：

> Optimized Translation (natural)   
> The best way to get started is **with** a simple example. Quill **requires** a DOM element to initialize, and the editor will be rendered **inside** that element. The contents of the DOM element will **serve as** the initial content of Quill.

修改点如下：

- "from a simple example" → "with a simple example"，with 更准确
- needs → requires，requires 更正式
- "at that DOM element" → "inside that element", 去掉冗余的 DOM, inside 更准确
- "will be the initial contents" → "will serve as the initial", 这个改动比较神

还有一个更口语的版本：

> Optimized Translation (informal)   
> The **easiest** way to get started is with a simple example. Quill **needs** a DOM element to **work**, and the editor will **show up** inside that element. **Whatever’s already in** the DOM element will **become** the initial content in Quill.

读起来确实流畅多了。

但是啊但是，这些相比原文档都长了很多——最关键的点就是这句 “Quill 需要用一个 DOM 元素进行初始化，用来承载编辑器”，在原文里就一句话： Quill is initialized with a DOM element to contain the editor, 但翻译都是对着中文结构，是 2 个子句。 如果把中文翻译成 “Quill 是从一个用来承载编辑器的 DOM 元素初始化的” 应该就更容易实现精确地反向翻译了；只不过这个中文句子就会显得有点拗口。

上面就是一个 inverse translation 的例子， 通过这种方法，是不是更能体会到中英地道表达的差异呢？对加深 Native English 的感觉应该是有用的吧 :) 大家可以多试试这种方法。

---

下面一段只有一句话

<section class="st">
  <section class="src-tag">“ Source Text</section>
And that's all there is to it!
</section>

能够猜到大概什么意思，但细究起来是不是感觉有点理解不了？我以前和公司财务对话就这个感觉——一个个都是很简单的词，但合起来，不知道 the exact meaning.

询问 ChatGPT, 这是一句非常口语化的表达，意思是 “事情就这么简单” / “就这些没别的了”，常用在教程或解释之后。

可以拆开成下面的部分来方便理解：

- that’s all：就这些
- there is：这...
- to it：关于这件事

当然，我们更应该直接去把整句背熟，在特定语境下争取脱口而出。

看下面的例句：

<section class="callout">
  <span class="hdot"></span> And that's all there is to it 例句

- You say "and that’s all there is to it!" to mean that something is very simple and does not require any more explanation or steps.

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

“体现在，来自于”。类似的词有 lies in, 但 lies in 更正式。come in 还有其他的一些常见含义，我们以后遇到了再说。


<section class="callout">
  <span class="hdot"></span> comes in 例句

- If something comes in a particular way or comes in + abstract sense, it can mean it exists or appears in a particular form or way, sometimes close in meaning to "lies in" or "consists in".
- The difficulty of the task comes in the details.
- The beauty of the design comes in its simplicity.
- The advantage comes in how easy it is to use.
</section>


<section class="kp">
  <span class="dot"></span>
get an idea of …
</section>

“了解 / 大致知道”。比 understand 更自然随意。看下面例句：

<section class="callout">
  <span class="hdot"></span> get an idea of … 例句

-  If you get an idea of something, you begin to understand it, have a general understanding, or form a mental picture.
- This tutorial will help you get an idea of how the software works.
- By looking at the diagram, you can get an idea of the system architecture.
- He explained the process so I could get an idea of what was happening.
</section>


<section class="kp">
  <span class="dot"></span>
play around with …
</section>

“玩 / 随便试”。非常口语化，比 experiment with 更轻松。看下面例句：

<section class="callout">
  <span class="hdot"></span> play around with … 例句

- If you play around with something, you try it, experiment with it, or use it in a relaxed way to see what it does.
- You can play around with the settings to see what works best.
- I spent the afternoon playing around with the new API.
- I’m just playing around with the new library to see what it can do.
</section>


<section class="kp">
  <span class="dot"></span>
head straight to …
</section>

“直接去 / 直奔”。非常地道口语，比 go directly to 更生动。

<section class="callout">
  <span class="hdot"></span> head straight to … 例句

- If you head straight to a place or section, you go there directly, without stopping or detouring.
- Head straight to the "Settings" tab to configure the options.
- Users can head straight to the login page from the homepage.
- Click the icon to head straight to the advanced options.
</section>

<section class="kp">
  <span class="dot"></span>
an in-depth walkthrough
</section>

“深入地**逐步**学习”。walkthrough 指 step-by-step guide or demonstration.
类似表达的有 a detailed guide, a step-by-step guide.

<section class="callout">
  <span class="hdot"></span> an in-depth walkthrough 例句

- An in-depth walkthrough is a detailed, step-by-step guide or demonstration that explains something thoroughly, often in software, games, or technical processes.
- The tutorial provides an in-depth walkthrough of the installation process.
- We prepared an in-depth walkthrough of the new feature for developers.
- This video offers an in-depth walkthrough of the software interface.
</section>

<section class="kp">
  <span class="dot"></span>
take a look at …
</section>

“看一看”。非常常用的口语，比 read/check 等都口语。

<section class="callout">
  <span class="hdot"></span> take a look at 例句

- If you take a look at something, you examine it, check it, or review it, often briefly or casually.
- Take a look at the new feature in the dashboard.
- I'll take a look at the code to see what's wrong.
- Please take a look at the documentation before you start.
</section>

## End

好了，QuickStart 部分就看完了，我们这期的内容也结束啦。短短的一篇文档，蕴含着这么多的地道表达，一定要多多练习，内化为自己的东西哈，与君共勉了。

喜欢我们的内容，就点赞收藏吧，咱们下期再见。