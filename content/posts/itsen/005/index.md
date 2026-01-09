---
title: "#005 IT 热点速递 | Chatterbox TTS Turbo, Sopro TTS, 基于 Rust+async 的嵌入式框架 Embassy"
date: 2026-01-09T10:37:59+08:00
slug: "itsen-005-chatterbox-turbo"
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

Hi, 各位读者朋友，欢迎回到 FS IT's English Channel. 这是我们的第 5 期内容。

本期我们来看 2 个 TTS 项目介绍和一个基于 Rust 的嵌入式框架。

## Chatterbox TTS Turbo

Made with ♥️ by RESEMBLE.AI.

github 地址：https://github.com/resemble-ai/chatterbox
HF 地址: https://huggingface.co/spaces/ResembleAI/chatterbox-turbo-demo (model-scope 上也可以找到，方便下载 model)

关注到这个是因为前段时间（所以这个也没有“速递”了 :)）看到的 tweet:

<section class="st">
  <section class="src-tag">“ Source Text</section>

This is the DeepSeek moment for Voice AI.

Chatterbox Turbo is an MIT-licensed voice model that beats ElevenLabs Turbo & Cartesia Sonic 3!

- <150ms time-to-first-sound
- Voice cloning from just 5-second audio
- Paralinguistic tags for real human expression

100% open-source.
</section>

这里的 *Paralinguistic*, 词典解释是 relating to communication through ways other than words, for example tone of voice, expressions on your face and actions，就是用来控制语调、语速、发声质量等的标记，如 `[laugh]`, `[cough]`.

这里介绍的是新发的 Turbo model，参数量 350M, 仅支持 English. Chatterbox-TTS 其实已经发布很久了（至少8个月），看下 Github 的描述：

<section class="st">
  <section class="src-tag">“ Source Text</section>

Chatterbox is a family of three state-of-the-art, open-source text-to-speech models by Resemble AI.

We are excited to introduce Chatterbox-Turbo, our most efficient model yet. Built on a streamlined 350M parameter architecture, Turbo delivers high-quality speech with less compute and VRAM than our previous models. We have also distilled the speech-token-to-mel decoder, previously a bottleneck, reducing generation from 10 steps to just one, while retaining high-fidelity audio output.
</section>

单词 *retain*, To retain something means to continue to have that thing, 保留、保持，比较正式的一个词。看下面几个例句：

<section class="callout">
  <span class="hdot"></span> retain 例句

- The interior of the shop still retains a nineteenth-century atmosphere.

  这家商店的内部装修仍然保留着19世纪的风格。

- If left covered in a warm place, this rice will retain its heat for a good hour.
  
  如果加盖放在暖和的地方，这些米饭可以保温足足一个小时。

</section>

我自己跑了下这个 model, 模型下载下来一共 3.8G; 在我的 3060 12G 上可以跑起来，速度也还可以接受。需要一个 clone 音源，我拿之前 kokoro 的音频作为音源，跑了下结果，可以听下对比：

Kokoro Output:

<audio controls>
  <source src="sage-kokoro.m4a" type="audio/mp4">
  Your browser does not support the audio element.
</audio>

ChatterTTS Turbo

<audio controls>
  <source src="sage-chatterbox-turbo.m4a" type="audio/mp4">
  Your browser does not support the audio element.
</audio>

这个例子来看，Chatterbox Turbo 效果不太好——主要体现在音质比较差。或许还是拿 TTS 的音频去做 clone 音源的问题。我之前测试它的 Demo，感觉它的结果更自然点，特别是配合上 paralinguistic tag, 不过在我这个设置下，确实没啥优势。

## Sopro TTS

葡萄牙的一个个人开发者作品。Github 地址 https://github.com/samuel-vitorino/sopro

<section class="st">
  <section class="src-tag">“ Source Text</section>

Sopro (from the Portuguese word for "breath/blow") is a lightweight English text-to-speech model I trained as a side project. Sopro is composed of dilated convs (à la WaveNet) and lightweight cross-attention layers, instead of the common Transformer architecture. Even though Sopro is not SOTA across most voices and situations, I still think it's a cool project made with a very low budget (trained on a single L40S GPU), and it can be improved with better data.

Some of the main features are:

- 169M parameters
- Streaming
- Zero-shot voice cloning
- 0.25 RTF on CPU (measured on an M3 base model), meaning it generates 30 seconds of audio in 7.5 seconds
- 3-12 seconds of reference audio for voice cloning

</section>


这个有点奇怪，听 demo 音频显然音质是很差的，但是 Hacker News 上倒是挺多赞的。
有一条评论是质疑的：

<section class="st">
  <section class="src-tag">“ Source Text</section>

I don't understand the comments here at all. I played the audio and it sounds absolutely horrible, far worse than computer voices sounded fifteen years ago. Not even the most feeble minded person would mistake that as a human. Am I not hearing the same thing everyone else is hearing? It sounds straight up corrupted to me. Tested in different browsers, no difference.

</section>

<section class="kp">
  <span class="dot"></span>
feeble
</section>

If you describe someone or something as feeble, you mean that they are weak. 


<section class="callout">
  <span class="hdot"></span> feeble 例句

- He's a bit feeble nowadays.
- a feeble old man.

</section>

feeble minded 字面指心智虚弱，也即愚蠢、智力极低的，是一个有点冒犯性的词语。

<section class="kp">
  <span class="dot"></span>
mistake A as/for B
</section>

If you mistake A as B, you wrongly believe that A is B, or think that A and B are the same thing.

常见变体：mistake A for B（更常用、更自然）

<section class="callout">
  <span class="hdot"></span> mistake A as/for B 例句

- I mistook him as a tourist because of his accent.
- From a distance, the object could easily be mistaken as a person.
- She was mistaken for someone else at the conference.

</section>

也许 Hacker News 上也有软文推广吧。不过这个项目确实还挺有意思的——对机器资源有限又想要训练一个 TTS 的朋友，可以参考看看。

## Embassy

项目地址：https://github.com/embassy-rs/embassy

一个用来写嵌入式程序的框架。

Embassy programs are compiled into firmware and flashed directly onto microcontrollers. There is no operating system — the code runs on bare metal, and Embassy provides its own async executor to schedule tasks after power-on.

基于 Rust + Async 来开发。


<section class="st">
  <section class="src-tag">“ Source Text</section>

The Rust programming language is blazingly fast and memory-efficient, with no runtime, garbage collector, or OS. It catches a wide variety of bugs at compile time, thanks to its full memory- and thread-safety, and expressive type system.

Rust's async/await allows for unprecedentedly easy and efficient multitasking in embedded systems. Tasks get transformed at compile time into state machines that get run cooperatively. It requires no dynamic memory allocation and runs on a single stack, so no per-task stack size tuning is required. It obsoletes the need for a traditional RTOS with kernel context switching, and is faster and smaller than one!

</section>

blazingly: blazingly is used to emphasize that something is extremely intense or impressive, especially in informal or technical contexts.

unprecedentedly: Unprecedentedly is used to say that something happens in a way or to an extent that has never happened before. 前所未有的

<section class="kp">
  <span class="dot"></span>
 obsolete 
</section>

/ˌɑːbsəˈliːt/，注意是 `iː`.

传统词典里只有形容词用法，Something that is obsolete is no longer needed because something better has been invented. 这里是动词，To obsolete something means to cause it to become outdated or unnecessary.

<section class="callout">
  <span class="hdot"></span> obsolete 例句

- Smartphones obsoleted standalone GPS devices.
- The system was obsoleted by a more efficient alternative.
- With technological changes many traditional skills have become obsolete.
- So much equipment becomes obsolete almost as soon as it's made.
  
  这么多设备几乎一生产出来就要被淘汰。

</section>

有相关需求的朋友可以看看这个，依赖简单，看起来编译好刷到固件里就可以用了。

## End

好啦，这就是本期的内容啦。我们在继续缩减投入量，所以现在就不放 TTS 音频了。
但是更新还是会继续的，所以 stay tuned for updates!
