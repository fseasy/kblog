---
name: hugo-multilang-blog
description: Hugo 多语言博客的文章组织方式，index.md 是默认语言版本
metadata:
  type: reference
---

# Hugo 多语言博客文章组织

## 核心规则

Hugo 多语言模式下，`DefaultContentLanguage: zh-cn` 时：
- `index.md`（无后缀）= 默认语言版本 → **中文**
- `index.en.md` = 英文版
- `index.zh-cn.md` = 中文版（显式标记）

## 避坑

之前遇到过 WARN: Search page not found 的问题，原因是：
- 把 `content/page/search/index.md` 当作英文版，但 Hugo 把无后缀的 `index.md` 当作默认语言（中文）
- 修复：把 `index.md` 重命名为 `index.en.md`，新增 `index.zh-cn.md` 作为中文版

## 相关文件

- 配置文件: `hugo.yaml`
- 仓库路径: `/Users/fseasy/workspace/dev-repo/kblog/`
