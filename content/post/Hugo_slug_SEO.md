---
title: hugo slug 与 SEO
slug: hugo-slug-SEO
filename: Hugo_slug_SEO
share: true
draft: true
date: 2024-06-18T21:47:30+08:00
tags:
  - hugo
categories:
---
# What is SEO?

Search Engine Optimization，搜索引擎优化，是指通过一系列优化技术和策略，提高网站在搜索引擎结果页面（SERPs）中的排名。<br>
*当然这里的优化技术不是某度的竞价排名。*

# What is slug?

"slug" 是一个术语，通常指的是一种友好的URL标识符。具体来说，slug 是用于标识一篇文章、页面或其他内容的一段文本字符串，通常是可读的，便于用户理解和搜索引擎优化（SEO）。<br>
slug 作为 url 的重要内容，我们还要对其进行优化：
* 避免停用词，如 "and", "or", "but", "the", "a" 通常不影响内容的理解
* 使用关键词

# slug in Hugo

设置 `permalinks` 启用 slug 作为 url。

```
[permalinks]
note = "/note/:year/:month/:day/:slug/"
post = "/p/:slug/"
```

在 Hugo 中，可以在每篇文章的 Front Matter 中设置 `slug`。<br>
如果 `slug` 没有设置，那么 hugo 会使用 `title` 作为 `slut` 的缺省值。<br>
通常我们设置一个 `slug` 之后，hugo 还会进行优化：
1. 转化所有内容为小写
2. 使用 `-` 替代空格
3. 删除所有特殊字符