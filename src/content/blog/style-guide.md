---
title: '排版样式指南'
description: '用来检验本站排版样式的文章：标题、列表、引用、代码、表格、图片与链接。'
pubDate: 2026-11-10
tags: ['设计', '测试']
draft: true
---

这篇文章用来检验博客正文的各种排版元素，确保样式覆盖完整。

## 二级标题带黑色左条

正文段落：包豪斯风格的要点是几何、功能与克制。本段故意写长一点，用来观察约 70 字符的行宽限制以及中文 1.8 倍行高的阅读感受，看看长时间阅读是否舒适、段落之间的间距是否恰当。

### 三级标题

这是一个 [带粗下划线的链接](https://astro.build)，以及一段 `inline code` 内联代码。

## 列表

无序列表：

- 第一项
- 第二项
- 第三项

有序列表：

1. 先做这个
2. 再做那个
3. 最后收尾

## 引用

> 形式追随功能。
> —— 一句适合放在这里的设计名言，用来检验拉丝银色左边条的引用样式。

## 代码块

```ts
// TypeScript 示例
interface Post {
  title: string;
  pubDate: Date;
  draft: boolean;
}

export function published(posts: Post[]): Post[] {
  return posts.filter((p) => !p.draft);
}
```

```bash
# Bash 示例
npm run build
npm run preview
```

## 表格

| 元素 | 样式 |
| ---- | ---- |
| 边框 | 2px 纯黑 |
| 圆角 | 无（除圆形外） |
| 阴影 | 硬偏移阴影 |
| 配色 | 纯黑 / 纯白 |

## 图片

<img src="/favicon.png" alt="站点图标" width="48" height="48" loading="lazy" />

## 分隔线

以上是分隔线之前的内容。

---

以上是分隔线之后的内容。指南到此结束。
