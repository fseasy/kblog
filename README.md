# Knowledge Blog

记录知识。

## 使用 Hugo Theme Stack

<img align="right" width="150" alt="logo" src="https://user-images.githubusercontent.com/5889006/190859553-5b229b4f-c476-4cbd-928f-890f5265ca4c.png">

Card-style Hugo theme designed for bloggers.

Use this template: [CaiJimmy/hugo-theme-stack-starter](https://github.com/CaiJimmy/hugo-theme-stack-starter)

尝试的折腾：

- 尝试代码自动换行 wrap：效果很差； 存在的问题： 1. 简单的wrap，代码wrap出来会把行号给撑开 2. 使用 grid 布局：行号不撑开了，但是换出来顶到头部的，和编辑器的体验还是差距巨大；而且和 highlight 高亮有冲突，高亮行会错乱。 ——结果就是，摸折腾了。

## 多语言文章组织

Hugo 多语言模式下，文章的组织和语言后缀规则：

**原则**：
- `index.md`（无后缀）= 默认语言版本（`DefaultContentLanguage: zh-cn`）
- `index.en.md` = 英文版
- `index.zh-cn.md` = 中文版（显式标记）

**示例：写一篇同时有中英文的博客**

假设你想写一篇名为 "Flutter 笔记" 的文章：

```
content/posts/flutter-notes/
├── index.md          # 中文版（默认语言）
└── index.en.md       # 英文版
```

中文版 `content/posts/flutter-notes/index.md`：
```yaml
---
title: "Flutter 笔记"
date: 2026-05-23
---
这是中文内容...
```

英文版 `content/posts/flutter-notes/index.en.md`：
```yaml
---
title: "Flutter Notes"
date: 2026-05-23
---
This is English content...
```

**注意事项**：
- 两版本的 `title`、`date` 可以不同
- `date` 需要各自指定（否则会使用文件创建时间）
- 两版本会自动关联，通过 `.Translations` 可以获取所有翻译版本