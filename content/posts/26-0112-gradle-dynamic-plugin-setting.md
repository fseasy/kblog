---
title: "Flutter Android build issue: Build was configured to prefer settings repositories over project repositories but repository 'injected_offline_android' was added by settings file 'settings.gradle.kts'"
date: 2026-01-12T13:30:20+08:00
slug: "26-0112-gradle-dynamic-plugin-conflict"
draft: false
image: 
math: false
license: CC BY-SA 4.0
comments: true
hidden: false

tags:
  - build-issues
categories:
  - flutter
---

<!-- 开始写作 -->

## Error

```bash
FAILURE: Build failed with an exception.

* Where:
Settings file '/home/fseasy/workspace/github/vox-echo/android/settings.gradle.kts' line: 20

* What went wrong:
Error resolving plugin [id: 'com.android.application', version: '8.11.1', apply: false]
> Build was configured to prefer settings repositories over project repositories but repository 'injected_offline_android' was added by settings file 'settings.gradle.kts'
```

## Solve Log

折腾了一天，ChatGPT、 Android Studio 和千问轮番指导，都是不行。在这种 hard 模式下，LLM 前后逻辑矛盾，气得骂人。
我尝试脱离这个项目，创建了官方 sample 项目，结果还是报这个错误——我就奇怪了，如果真是 flutter 的问题，那不得是 P0 bug?
尝试搜索了下，发现一个 Issue 是说回退 gradlew 的版本，回到 8.1.1; 整啊，还换了 java 版本，最后发现又报了 file reference 的问题，
傻了。LLM 说还是版本问题，心凉。

最后又去 flutter issues 里搜 ，"injected_offline_android" 这个一点都没有；" Build was configured to prefer settings repositories over project repositories" 这个还有比较新的——终于啊终于，找到了 [Android build fails if environment contains global repositories defined via allProjects.repositories ](https://github.com/flutter/flutter/issues/174035).

真正原因是：


> I found that build fails when system has global init.gradle file with repositories defined via allProjects.repositories (in our case we need it for other projects to be able to access internal Artifactory) due to the RepositoriesMode set to FAIL_ON_PROJECT_REPOS in flutter_tools package.

总结下：

1. 有一个全局的 init.gradle, 把当前项目 gradle 给做了 Inject 处理
2. `flutter_tools` 又设置了 RepositoriesMode 为 FAIL_ON_PROJECT_REPOS；导致我们在 setting.gradle.tks 里自己再设置，根本无效 （LLM就是说要设置这个！但是一直无效，原来是被这个工具预先给设置了）

解决方法，人家也说得很清楚:


> I have a work around that I do not like very much, but it does get me unblocked for the current moment. This is in `~/.gradle/init.gradle`.
> 
> ```kotlin {linenos=inline hl_lines=[10] style=emacs}
> // Check if this is a Flutter project
> def isFlutterProject = false
> def currentDir = gradle.startParameter.currentDir ?: new File(System.getProperty("user.dir"))
> def currentDirPathLowerCase = currentDir.absolutePath.toLowerCase()
> if (currentDirPathLowerCase.contains("flutter") || currentDirPathLowerCase.contains("android")) {
>     isFlutterProject = true
> }
> 
> // Skip allprojects repository configuration for Flutter projects
> if (!isFlutterProject) {
>     allprojects {
>         repositories {
>             ...
>         }
>     }
> }
> ```

我就去我的 `~/.gradle/` 下去看，还真有个,在 `~/.gradle/init.d/offline.gradle`, 内容是

```kotlin {hl_lines=[11,20] anchorLineNos=true}
def reposDir = new File(System.properties['user.home'], ".android/manual-offline-m2")
def repos = new ArrayList()
reposDir.eachDir {repos.add(it) }
repos.sort()

allprojects {
  buildscript {
    repositories {
      for (repo in repos) {
        maven {
          name = "injected_offline_${repo.name}"
          url = repo.toURI().toURL()
        }
      }
    }
  }
  repositories {
    for (repo in repos) {
      maven {
        name = "injected_offline_${repo.name}"
        url = repo.toURI().toURL()
      }
    }
  }
}
```

好哇好哇，原来如此——这个 `injected_offline_android` 果然是你给插进去的——想起之前 LLM 说去 Android Studio 里关掉 offline, 还是有道理的！
只是我也关了，却没用——这里看来根本没有修改这个 config 啊。

然后我想，就这 1 个 issue 报错，说明这个不是共性问题，而是我自己机器的问题啊？

看了下这个文件，原来是 2021 年就创建的？？真是时间的玫瑰，现在给绽放了。gradle 不知道迭代多少轮了，这中动态 inject 的，早就被抛弃了吧……
得，可算知道原因了，也不用写 if 了，直接给你删掉吧。

重新编译，这个 `injected_offline` 报错没了，但还是报了别的错——可能是我现在环境搞乱了（之前还装了 fvm 来降级 flutter）。得，回滚环境吧！
于是 fvm 准备装最新的——装的过程中，我手贱又开了 vscode，打开 flutter 插件就开始自动更新啥的——反正一会儿鼠标就动不了了。
可恶，又被这个系统恶心到了。

直接强制关机，明日再战吧。

次日重置环境，发现 fvm 安装的环境并不能改变全局的 flutter ——那我还要你干嘛——直接卸载了 fvm；
项目里删掉 android 重新 create, 结果发现下了好多 android sdk 的问题——估计是 java 版本问题？
忘了切回来了——赶紧改了 flutter jdk home, 指向了 Android sdk 用的 jdk, 再次编译。
怎么着——不到1分钟，apk就出来了。

哎——如此丝滑的流程，硬是被我搞了这么久。

## 经验总结

1. gradle 这个报错太不清楚了——就说 'injected_offline_android' was added by settings file 'settings.gradle.kts'**——但是我这个 settings.gradle.kts 根本没有这个啊？作为新手，完全不知道它会被 home 目录的 gradle 给控制——或者说，控制没关系，你倒是把报错信息搞得更指向清楚些啊。第一个锅，扣你头上。
2. LLM: 其实都说到了核心问题，只是——它也不知道 gradle、flutter 的机制，所以来回倒腾，解决不了问题。还让人觉得跟它对话太愚蠢！
3. 自己好浮躁：其实看几个 issue 就可以快速解决的。自己太依赖 LLM 了，对 issue 这种交互式、信息密度低的，缺乏耐心。

LLM 出来的结果，只能解决老旧问题、解决不了新问题。减少对它的依赖，老老实实搜 issue 更好！