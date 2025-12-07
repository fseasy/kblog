---
title: "000: "
date: 2025-12-05T16:55:51+08:00
slug: "its-en-000"
draft: true
image: 
math: false
license: CC BY-SA 4.0
comments: true
hidden: false

tags:
  - 公众号
categories:
  - FS-ITs-English
---

Hello, 各位读者和听众们，欢迎来到 FS IT's English Channel. 这是频道的第零期内容，所以先简单介绍下这个频道的定位、动机和内容构成。

- **定位**：我们预期面向的受众是有英语学习需求或动机的 IT 、科技行业从业者。但任何有兴趣的读者听众我们都是欢迎的。
- **动机**：我本人英语并不好，之前有幸出国工作一年半。作为社恐，自己主要就是通过看别人写的句子 + ChatGPT 解释扩展 + 重复跟读来学习的英语。这肯定不是高效的方法，但我觉得应该适合边学边用、不着急应试、只有碎片时间的群体。现在我辞职回国了，不想落下英语，也想开始搞自己的品牌，所以就弄了这个频道。
- **内容**：
  - 数据源是一手的英语 IT、科技新闻 & 知识；目前看应该会包括：
    - Hacker News, TechChurch, The Verge X, Reddit 等
    - 技术 Docs/Papers
    - Github 上的开源项目等
  - 加工流程包括：
    - 筛选 (curating) 出有英语学习价值的单词、句子和段落；
    - 基于 AI 做一些重点解释和扩展；
    - 配上用于收听的全文音频；
    - 提供用于练习的资料：更结构化的英语部分文本，以及对应的 TTS 音频。
  - 版块设计：现在没想好，可能一开始就零碎的堆砌到足量内容就行。
  - 更新频次：先保证一周 3 更，坚持 1 年再说。
- **盈利模式**：应该就是常规的广告模式吧，不过小（长期）目标的还是积累自己的品牌，方便后续推销自己开发的应用。扯太远了，先积累用户吧，养肥了再“杀”哈哈。

好了。这就是本频道的一个 config 了，下面开始第零期的内容吧。

本期内容围绕 Hacker News "Cloudflare Down Again" 的话题，先看看 Hacker News 上的口语化评论，接着看下 Cloudflare status 页面上的书面描述，最后再来学习下 best practices for status pages.

## Part1: (Hacker News) "Cloudflare Down Again"

先看标题 "Cloudflare Down Again"，它并非一个完整句子——只有主语 Cloudflare、形容词 Down 和副词 Again, 缺了谓语。可以补一个 "is" 句子就完整了。这种省略谓语系动词(linking verb, be verb)的句式，在标题这个位置是非常常见的，或者说它就是叫做 headline-style，在推文、口语里也是很常见的。

> Q（小白）: 那么，Down 可以看做一个动词吗？  
A（ChatGPT）: 不，down 虽然有动词用法，但一般表示击倒、放倒(e.g: He downed the opponent in 10 seconds.) ，用得很少。这里做省略谓语的形容词讲更合适。

> **扩展1**：当然，除了可能是 *is down*, 也可以是 *went down*. 注意，is 是现在时(present tense), went 用过去时 (past tense).

> **扩展2**：接着*扩展 1*，那这里省略的可能是 went 吗？答案是不太可能。因为英语标题默认只能省略系动词 be，而不能省略实义动词（如 went）。

OK，标题看完，我们点进去看评论，这条评论挺有意思，我们拉出来：

> Yes. Weird that https://www.cloudflarestatus.com/ isn't reporting this properly. It should be full of red blinking lights.

第一个点是 `weird` 这个单词，是 “奇怪的” 意思，collins 释义 *very strange or unusual and difficult to explain*. 这个单词在我出国以前基本闻所未闻，但过去后发现这个词在 slack、论坛等口语场景里用的频率极高，远超 *strange* 这个词。这个实际感受，与 collins 词频统计是不符合的——在 collins 词频统计里，weird 是 2 星，但 strange 却有 4 星。其原因大概率是统计样本不一致的问题：collins 统计的主要是书面语词频，不能很好反应口语、网络用语的频次。而 weird 恰恰是在现代互联网阶段（2010~）使用量暴增的。不过好在现在有吃了全网数据的大模型在，我们一定程度上可以规避这种偏差了。

第二点是关于 status page 更新的问题，主评论是说 status page 没有反应当前宕机的情况，这是超出预期的。下面的评论就比较现实了，摘 2 个出来看看：

- *Yeah. I only work for a small company, but you can be certain we will not update the status page if only a small portion of customers are affected, and if we are fully down, rest assured there will be no available hands to keep the status page updated.*
   
  这个评论是有点搞笑的，或者说有点阴阳了。

  重点看 2 个地方：

  1. *you can be certain*: 表示“你完全可以确定/毫无疑问”。 certain 是“确信的”含义。类似的可以用 *you can be sure*, *you can bet*(更口语), *I guarantee*.
  
  2. *rest assured*: 这是英语里常用的固定表达，意思是“请放心/尽管安心”。 *rest* = 放松、安心；assured = 得到保证的，有把握的；合起来就是一种“让对方安心”的表达。近似的表达是 *don't worry*, 但 *rest assured* 语气更正式、坚定。下面是 2 个例句：

    - *Rest assured,* we are handling the situation.

    - *You may rest assured* that no data was lost.

- *It’s wild how non(e) of the big corporations can make a functional status page.*
  
  这里 2 个单词值得一说：

  1. *wild*, 表示“离谱/夸张/不可思议/难以想象”。同 *weird*, *wild* 也是口语里常用的词语，其语气既强烈又随意，看下下面的常见用法：

    *That's wild.*: 一个短句，表达“太离谱”的感叹。

    *He wen wild.*: 情绪化/夸张的语气，说“他疯了”。

    *The party was wild.*: 积极、刺激的语气，“很嗨”的意思。

  2. *functional*，这里是“起作用的/运转正常的”的意思。*Functional equipment works or operates in the way that it is supposed to.*

    这个单词算是 IT 场景种常见的，如 *Is the API functional now?*, *functional issues*, 但口语中，更常用 *Is it working now?* 这种表达。

  这条评论的子评论也有点犀利：

  *They could, but accurate reporting is not good for their SLAs.*

  做云服务相关的同学对这个 *SLA* 肯定不陌生，它是 *Service Level Agreement* （服务等级协议）的缩写(acronym)。SLA 通常包括可用性保证、响应时间、服务恢复时间和故障赔偿条款。


好了，评论就看到这里。


## Part2: Cloudflare status page content

点开前面评论提到的 [cloudflare status](https://www.cloudflarestatus.com/), 我们看到如下内容：

![cloudflare-status-page-screen-short][cloudflare-status-page-screen-shot]

先来看看标题栏：**DTW (Detroit) on 2025-12-05**，这个 *DTW* 是什么意思？

Detroit 就是底特律，DTW 是 Detroit Metropolitan Wayne County Airport 的 IATA (International Air Transport Association)机场代码。原来，Cloudflare, AWS 等全球网络服务商，都用机场代码来代表机房/数据中心。IATA 代码是唯一的、可无歧义查询的 3 个字母，的确很适合这种全球化的 IT 服务商。犹记得我当时在百度时，应该还是用的拼音缩写来表示机房位置，如 nj01, bj02 这种，感觉还是有点歧义的。


再看两个加粗的内容：*In progress*, *Scheduled*, 都是表示状态。 *In progress* 表示“进行中”，*Scheduled* 表示“已安排，待执行”。这两个词语在项目管理中比较常见；特别的，*WIP (Work in Progress)* 是描述工作进度的常见缩写。

接着看描述：

- *Scheduled maintenance is currently in progress. We will provide updates as necessary.*
  *maintenance* 这个单词是“维护”的意思，在 IT 场景也极为常用。除了上面 *scheduled maintenance* 表示的计划性维护, 还有一些常见的场景：

  - *The server is down for maintenance.*
  - *The site is currently in maintenance mode.*
  - *This is a maintenance release focusing on stability.*
  - *This code has a high maintenance cost.*

  Maintenance 对项目是很重要的，然而它的价值却往往被轻视了。这和扁鹊治病有点像，维护得好不出问题，体现不出价值；等挂了连夜 on call 修复，往往才体现出功/苦劳来。啊哈，扯远了。

- *We will be performing scheduled maintenance in DTW (Detroit) datacenter on 2025-12-05 between 09:00 and 13:00 UTC.*
   
  经典句子，工作中肯定会用到类似的，背熟它没毛病。
  
  特别注意这个 *will be doing* 结构，是将来进行时(Future continuous)，这用得也非常多。相比于一般将来时(Simple Future), 这种用法更常用在计划的持续性活动中，给人一种“预先通知/准备”的感觉，比一般将来时更礼貌、柔和。看看工作中常用的例子：

  - *During the meeting, I will be presenting the new project.* （安排/告知工作）
  - *OOO(out of office) next Friday. I will be attending a family event.* (请假解释)

- *Please make sure you are expecting this traffic to fail over elsewhere during this maintenance window as network interfaces in this datacenter may become temporarily unavailable.*
   
  比较典型的长句，使用 *as* 连接后面的原因状语从句；可以用 *because* 来替换 *as*，但书面中 *as* 会更加正式一点。

  从 *as* 拆开，前面部分最关键的短语是 *fail over*, 它是一个固定短语，在 IT 网络运维里比较常用（可以直接合起来，即 *failover*），表示“故障切换/备援切换”，连起来整个意思就是“请确保你预期流量会自动切换到其他地方”。即便把“故障切换”换成了“自动切换”，翻译成中文还是怪怪的，因为中文里不太会这么组织语言。但反过来说，我们也从中学会在这种场景下，英语该怎么正式表达。

  *as* 后面部分，说下 *temporarily* 这个单词，是“临时地”意思，对应的形容词是 *temporary*. 计算机里常说的“临时文件”、`tmp`, `temp` 就对应这个单词。

OK，看到这里为止。看起来今天是有 Scheduled Maintenance, 不过 Hacker News 上说的问题，应该不是这个维护导致的吧？毕竟里面说到流量会自动 failover 的。看来状态页还真没更新啊？


## Part 3: Best Practices for Status Pages

围绕构建 Status Pages 的主题，我们最后来到 openstatus.dev 的 [Best Practices for Status Pages](https://docs.openstatus.dev/concept/best-practices-status-page).

文章开篇论述了 Status Page 的重要性，并引出了后续的最佳实践。这段文本写得挺漂亮的，建议全文背诵，工作中用得上。

> Status pages are essential tools for communicating service health and incidents to users. They build trust by providing transparency and should be a core component of any software company’s communication strategy. Following a set of best practices ensures your status page is effective and helpful.

这段话可以说 2 个点：

- 其一是 "transparency" 这个单词，是“透明性”的意思，*the quality of sth, such as an excuse or a lie, that allows sb to see the truth easily*.
- 其二是 *Following ...* 的句法结构，*Following a set of best practices* 是动名词短语(Gerund Phrase)做主语，*ensues* 是谓语，而后面整个是一个宾语从句。
  这里 *ensures* + 宾语从句，省略了 *that*. 这是一种常见做法，能让语气更自然。
  不过注意不是所有动词都可以省略 *that*, 省略前提是不引入歧义。复杂的咱搞不清，咱先记住类似的 *say, think, believe, know, hope* 这些动词是可以省略的就行。

继续看具体实践部分，分成了 4 大块，咱就不复制粘贴了，把关键的信息挑出来如下：

1. Maintain Transparency and Honesty

   *Be upfront about issues, even minor ones. Hiding problems erodes user trust and can lead to more support requests.*
  
   2 个点，

   - *upfront* 是“坦率的/诚实的”意思，也有“预付的”含义；

     *There will be an upfront fee of 4%.*

   - *erode* 表示“削弱（权威、权利、信心）”，是从基础的“（岩石/土壤）侵蚀引申出的。

     *Once exposed, soil is quickly eroded by wind and rain.* 一旦暴露在外，土壤很快就会被风雨侵蚀。

2. Automate Where Possible
   
   *Manual updates during an outage are prone to error and can be slow.*
   
   *outage* 一般指“停电”，这里用的是 *a failure or interruption in use or functioning* 含义，直观理解就是“服务挂了”。

   *prone* 表示“有……（不好的）倾向；易于……的”，也是很常见的单词。还可以用 `-prone` 后缀与名词连用构成形容词，如这里可以变成： *Manual updates are during an outage error-prone.*

3. Provide Detailed Incident Communication
   
   Let's check the examples below:

   - *Investigating: "We’re currently investigating an issue affecting user logins."*

     *investigating* 用来表示事故正在调查中。

   - *Identified: "We’ve identified the root cause as a database connection issue and are working on a fix."*

      *identified* 说明问题已经找到了！

      *root cause* 指问题的根源(root)，比用短语。

   - *Monitoring: "A fix has been deployed, and we’re monitoring the system’s performance."*

      *monitoring* 说明一个修复已经部署，正在监控当前状态。

   - *Resolved: "The issue has been resolved, and all services are now operational."*
      
      *resolved* 指问题已经被解决，这在 IT 场景里是一个固定状态词。不过在词典里，它基本只有“下定决心的”这个含义，和 *determined* 接近，但更正式、书面。例句如下：
      
      *I was resolved not to see him.*
      
      *operational* 在 status page 里也是一个常见的固定状态单词，表示“正常运行”。和前面提到的 *functional* 相比的话，前者表示“整体运行正常”，后者描述“功能可用”。
    
   这一小节其实就是说要把当前的服务状态、修复进展尽可能详细的告知客户。
   
   > 知识卡：Status Page 常见的固定状态词  
   Operational, Maintenance, Degraded, Outage, Investigating, Identified, Monitoring, Mitigated, Workaround, Resolved, Closed.

   前面基本都讲过了，补充说下下面的几个：

   *degraded*: 服务退化，性能下降。
   *The API is currently degraded due to high traffic.*

   *mitigated*: 问题已缓解（但没完全修好）。
   *Mitigation is in place while we work on the permanent fix.*  在我们开发永久修复方案期间，缓解措施已经生效。

   *workaround*: 应急方案，临时解决方法。
   *We provided a temporary workaround while engineering investigates.*


4. Make it Easily Accessible
   
   让你的 Status Page 能轻松被找到。提供了 2 个建议：

   - *Prominent link: Place a link to your status page in your application’s footer, on your website, and in your support documentation.*
     *prominent* 这里指“显眼的，容易被注意到的”。还有一个意思是 *重要的；杰出的；著名的*。

     *The church tower was a prominent feature in the landscape.*
  教堂的尖塔曾经是此地景观的重要特色。
     
     *She was prominent in the fashion industry.* 她曾在时装界名噪一时。

   - *Custom Domain: Use a simple, memorable URL like status.yourcompany.com.*

好啦，这份 Best Practices 就大概介绍完了。

Status Page 目前基本都是自动检测+人工维护的混合方式。自动的实时，但容易误报，无法细化问题，更无法同步修复情况；所以人工介入是必然的。一般来说，流程大概是后台自动监控 -> 有问题触发预警 -> page on-call, 然后相关人员就不得不开始放下手头的事情（或者没做完的梦 :)）来定位问题、拉群、打电话、请示领导、对外同步进展…… 

Hacker News 吐槽 Cloudflare status page 状态更新迟了，在业界估计也算正常吧。毕竟影响力这么大的服务，问题肯定很快会暴露给用户；而公司内部，大概率已经行动了起来，只是这种状态可能还未反应到 Status Page 上。

## END

好了，今天的内容到此结束。围绕 Status Page, 我们学习了口语和书面的一些表达，对其运转方式也有了简单的了解。

内容都可能有疏漏和错误，还请专业人士多多修正补充吧。下期再见。

*注：本内容大量使用了 LLM、Collins/OALD 等资源。致谢。*

<!--- CONTENT END --->

[cloudflare-status-page-screen-shot]: ./cloudflare-down.desc.webp