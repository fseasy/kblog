---
title: "#004 IT 热点速递 | Mockito TimvdLippe: Stepping down as maintainer after 10 years"
date: 2025-12-29T22:37:59+08:00
slug: "itsen-004-mockito-timvdlippe"
draft: false
image: 
math: false
license: CC BY-SA 4.0
comments: true
hidden: false

tags:
  - "时效热点"
categories:
  - "IT's English Channel"
---

Hi, 各位读者朋友，欢迎回到 FS IT's English Channel. 这是我们的第 4 期内容。

今天我们摘录部分 Mockito ("The most popular mocking framework for Java") 核心维护者 TimvdLippe 宣布卸任的原文，
学习下当我们要选择离开某个职位时，该如何贴切表达吧 :)

source：[Stepping down as maintainer after 10 years](https://github.com/mockito/mockito/issues/3777)

## Stepping down as maintainer after 10 years

首先我们看下短语

<section class="kp">
  <span class="dot"></span> Step down
</section>

If someone steps down, they leave an important job or position, especially one of responsibility.

“辞去某个职位，卸任”。常用于主动让位、平稳交接的语境。反过来，step up 就是指上任、承担更重要职责。

step down 后面常用介词 as 或者 from，as 后面常接具体的身份/角色；from 后面除了接职位，还可以是组织等。

<section class="callout">
  <span class="hdot"></span> Step down 例句

- The CEO stepped down after ten years in charge.
- She announced that she would step down as project lead at the end of March.
- He stepped down from leadership after the transition.
- After the scandal, the minister was forced to step down from office. (scandal, 丑闻)
</section>

接下来看正文，首先是离任的声明。

<section class="st">
  <section class="src-tag">“ Source Text</section>

In March 2026, I will be Mockito maintainer for 10 years (nearly a third of my whole life). Looking ahead, I decided that a decade milestone is a good moment to pass on maintainership to other folks. In the coming months until March, I will spend time ensuring a smooth transition in maintainership.

In this issue I list several considerations why I made the decision. Communication and discussion of plans for future maintainership will be somewhere else, most likely in a separate GitHub issue. Stay tuned for that.
</section>

```

关注 2 个短语。

<section class="kp">
  <span class="dot"></span> pass on A to B
</section>

If someone passes on something to another person, they give it to them so that the other person can take responsibility for it or continue with it.

“交接”。另一个常用的表交接的词是 hand over；要做对比的话，可以认为 pass on A to B 强调传承这种略虚的东西; hand over 偏重具体的事情或者流程本身。看下面的例句：

<section class="callout">
  <span class="hdot"></span> Step down 例句

- After a decade, he chose to pass on the torch to the next maintainers.
- She felt it was time to pass on her knowledge to the next generation.
- The maintainer is preparing to hand over the codebase.
- The founder handed over control of the company to a new CEO.
</section>

最后，注意 "pass on" 本身有“拒绝”的含义，如 I'll pass on the offer (我不考虑这个提议).

<section class="kp">
  <span class="dot"></span> Stay tuned
</section>

If you say stay tuned, you are telling people to continue watching, listening, or paying attention because more information will be given later.

“敬请关注”，应用非常广泛的词语。看下面的例句是否很熟悉：

<section class="callout">
  <span class="hdot"></span> Step down 例句

- More details will be announced soon — stay tuned.
- We’re working on the next release. Stay tuned for updates.
</section>

## Energy drain because of JVM agent change

上面 2 段 Tim 说了决定，接下来就将展开 3 点理由。限于篇幅，我们就摘第一点来看。

在看第一点前，先补充下背景（据 ChatGPT）：
Mockito 主要用 ByteBuddy 库修改底层字节码来实现 Mock；但是 JVM 22 出于安全考量限制了普通包(artifact)的字节码修改能力；
要想实现之前的类似能力，必须得编译成 Agent 类型的包。但 JVM 对 Agent 类型的支持尚不足够完善，配套工具链也没有完全建立。
也就是说，Mockito 要适配这种变更，不得不去踩坑！

基于此背景，我们先看第一点的前半部分：

<section class="st">
  <section class="src-tag">“ Source Text</section>

As you might know, Mockito 5 shipped a breaking change where its main artifact is now an agent. That's because starting JVM 22, the previous so-called "dynamic attachment of agents" is put behind a flag. This change makes sense from a security point-of-view and I support it.

However, the way this was put forward to Mockito maintainers was energy draining to say the least. Mockito is probably the biggest user of such an agent and is often looked at for inspiration by other projects. As such, Mockito often pioneers on supporting JVM features, built on a solid foundation with ByteBuddy. Modules was such a feature that took months of hard work by Rafael to figure out, including providing feedback to JVM maintainers.

Unfortunately such a collaborative way of working was not the case when discussing agents. To me, it felt like the feature was presented as a done deal because of security. While dynamic attachment is problematic in many ways, no alternative solutions were proposed. That's okay, as Mockito pioneers on these solutions, yet in this case I felt we were left alone.

</section>

先看单词

<section class="kp">
  <span class="dot"></span> drain
</section>

非常重要的一个单词。动词形式有排空、使耗尽的意思；名词形式有下水道、排水管、消耗的含义。这段里都是和 energy 搭配，用了 2 个形式：

- Energy drain: 做名词，消耗
- (be) energy draining，现在分词做形容词

实际上 energy drain 或者 energy-draining 都是一个固定搭配，An energy drain is something that makes you feel very tired, either physically or mentally, because it requires a lot of effort or causes stress.


<section class="callout">
  <span class="hdot"></span> Energy drain / Energy-draining

- Constant meetings have become a real energy drain.
  
  没完没了的会议成了真正的精力消耗。

- Maintaining the project alone was an energy drain over time.
  
  长期独自维护这个项目逐渐消耗了他的精力。

- The process was slow and energy-draining.
- Long-term maintainership can be energy-draining.
  
  长期的维护工作可能非常耗费精力。
</section>

再看下面这个句子：

> the way this was put forward to Mockito maintainers was energy draining to say the least.

句子核心是 the way was energy-draining, this was put forward to Mockito maintainers 是后置定语从句，修饰 the way, 省略了关系代词 that/in which.

其中两个重点短语：

<section class="kp">
  <span class="dot"></span> put forward
</section>

提出、提议. 常见于技术提案、标准讨论等正式场景。 If someone puts forward an idea, suggestion, or plan, they formally suggest it for consideration or discussion.

<section class="callout">
  <span class="hdot"></span> put forward

- She put forward a proposal to change the review process
- Several options were put forward during the discussion.

</section>

<section class="kp">
  <span class="dot"></span> to say the least
</section>

固定短语，“至少可以说”、“往轻了说”（实际更严重）。在这句话里是一个表语气的插入语，一般放在句子末尾，正式书面语境下前面会加一个 dash 或者逗号与主句隔开。
You use to say the least to emphasize that a statement is an understatement, and that the real situation is stronger or more extreme than you have said.

<section class="callout">
  <span class="hdot"></span> to say the least

- The last few months have been difficult, to say the least.

  最近几个月很困难——实际情况更糟。

- Running the project alone was demanding, to say the least.

  独自运行这个项目非常吃力——远不止如此。

</section>

最后列出其他比较重要的短语：

- so-called: 所谓的
- make sense: 有道理（高频口语）
- pioneer: n, 先锋；v, 当开拓者
- a done deal: 已经板上钉钉的事 (比较口语化)。*Once the contract is signed, the promotion is a done deal. 合同一签，升职就板上钉钉了。*
- yet (in this case...): 但是。转折显得克制但带失望——“按理说不该这样”；对比: but, 转折强度普通、中性；however, 正式、书面，偏理性。

继续看着点的后半段：

<section class="st">
  <section class="src-tag">“ Source Text</section>

My personal take is that folks involved with the change severely underestimated the societal impact that it had. The fact that proper build support is non-existent to this day shows that agents are not a priority. That's okay if it isn't a priority, but when it was communicated with Mockito I perceived it as "Mockito is holding the JVM ecosystem back by using dynamic attachment, please switch immediately and figure it out on your own".

Here, the fact that I (and others) are volunteers doing their best for the project, is important to understand the societal impact. When you put individuals under pressure, who do this work in their own time out of goodwill, things crumble. It's commonly joked about with XKCD's on the fact that the whole open source world relies on a couple of individuals. That couldn't be more true in this situation, where the collaborative system collapses when too much pressure is put on individual folks.

This saga planted the seed to reconsider my position as maintainer.
</section>

这里我们先看 2 个重点词汇/短语：

<section class="kp">
  <span class="dot"></span> hold back
</section>
 
这里是“阻碍，拖慢”。If something holds someone back, it prevents them from making progress or developing. 还有一个意思是“克制自己不去做某事”。

<section class="callout">
  <span class="hdot"></span> hold back 例句

- I don't want my availability to hold the project back.
- Outdated processes are holding the team back.
- She held back her anger during the meeting.
</section>

<section class="kp">
  <span class="dot"></span> crumble
</section>

碎裂、崩溃、瓦解。If something crumbles, it breaks into small pieces, or begins to fall apart. If a system, organization, or feeling crumbles, it suddenly or gradually fails or loses its strength.

<section class="callout">
  <span class="hdot"></span> crumble 例句

- The old stone wall began to crumble.
- The biscuit crumbled in his hand. (biscuit /ˈbɪskɪt/, 饼干)
- Her confidence crumbled after the setback. (setback, 挫折)
- The system began to crumble under pressure.
</section>

再看最后一句话：

> This saga planted the seed to reconsider my position as maintainer.

比较漂亮。学习 2 个点：

<section class="kp">
  <span class="dot"></span> saga /ˈsɑːɡə/
</section>

原意是“长篇史诗”，这里指“持续很久、反复拉扯、让人 energy-draining 的一整套事情”。 A saga is a long and complicated series of events or experiences.

<section class="callout">
  <span class="hdot"></span> saga 例句

- After all the delays, the saga of getting the project approved finally ended.
- The saga of our move across the country took months to resolve.
  
  我们跨州搬迁的“史诗级”折腾，耗时数月才尘埃落定。

</section>

<section class="kp">
  <span class="dot"></span> plant the seed (of/for/to) ...
</section>

种下...的种子。中文里也很常用，在英语里也是一个地道的表达。You plant the seed when you start an idea that may grow in the future.


<section class="callout">
  <span class="hdot"></span> plant the seed (of/for/to) 例句

- Her question about sustainability planted the seed of doubt in the team’s mind about the current design. (sustainability, 可持续性)
- The workshop didn’t propose a solution — it simply planted the seed to rethink how we measure success.
- Early exposure to open-source projects can plant the seed of curiosity that grows into a lifelong career.
</section>


最后简单列出其他重点词汇：

- societal: /səˈsaɪətl/ 社会的。与 social 做对比，societal 强调社会“制度”的，social 更多指“人际、社交”层面的。
- perceived: 察觉到的，强调主管感受。扩展词: perception(感知), perceptron(感知器).
- out of goodwill: 出于善意


## Summing it up

最后我们摘录 Tim 总结部分的一段如下：

<section class="st">
  <section class="src-tag">“ Source Text</section>

While these points had impact on me as maintainer, my hypothesis is that it doesn't apply to others in the same way. I know others are eager to work on Kotlin support for example. That's why I concluded that a decade is enough time to have helped Mockito forward. Now it's time for somebody else to take over, as I believe that's in the best interest of Mockito as a project. Because ultimately that's why I chose to become maintainer in the first place: I believed that with my work, I could improve Mockito for millions of software engineers.

</section>

就重点说 1 个短语：

<section class="kp">
  <span class="dot"></span> in the best interest of ...
</section>

对 ... 最有利，符合 ... 的最佳利益。If something is done in the best interest of a person or group, it is done to benefit them most effectively

<section class="callout">
  <span class="hdot"></span> in the best interest of ... 例句

- The doctor made the decision in the best interest of the patient’s health.
- We postponed the release in the best interest of product quality and user safety.
- The court ruled that placing the child with her grandmother was in the best interest of the child.
</section>

再来翻译最后一句话：Because ultimately that's why I chose to become maintainer in the first place: I believed that with my work, I could improve Mockito for millions of software engineers.

ChatGPT 翻译：因为归根结底，这就是我当初选择成为维护者的原因：我相信，通过我的工作，我能够让数百万软件工程师受益，并改善 Mockito 项目。

抉择时刻，记得回望初心啊。

## End

好啦，这就是本期全部内容了。

喜欢我们的内容，记得点赞评论和关注哦，下期再见!

