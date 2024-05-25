---
title: Astro 博客教程
published: 2024-05-25
description: 如何使用 Astro.js 搭建个人博客，以及一些杂谈。
tags: [Astro, Blog]
category: Tutorials
draft: false
---

# Astro?

在 Astro 之前已经有很多优秀的静态网页生成系统了，比如说 Github Pages 用的 Jekyll (Ruby)、Hexo (JS)、Hugo (Go)、VuePress (Vue)、Gatsby (React) 等等。作为一个写博客的人，我们唯一在乎的就是在 posts 文件夹里创建一个 Markdown，写点内容和 Frontmatter，然后等着这些工具把它们转换成 HTML。

既然如此，why Astro?

一大原因可能是我的强迫症。我习惯于给每个编程语言标签化，比方说 Python 就该是机器学习、C/C++ 就应是操作系统级别的开发、Go 就该是高并发网络编程等等。而 JavaScript 这门语言在我眼里（实际上也是）跟前端脱不了干系。所以选择 Astro 的主要因素仅仅是它是 JavaScript 框架。

我并不是讨厌 Vue 和 React。相反，这两个框架我都常用：在搭 SPA 的时候我会用 Vue 或者 Nuxt，在写移动端 UI 的时候我会用 Next.js + Tauri，这两个框架的扩展和 UI 库多到足以使我选择恐惧。当然这两个框架也提供了静态生成工具：VuePress 和 Gatsby。但是这两个框架的生态对于一个小小几乎纯静态的博客来说实在是小题大做了，我并不想把它们的组件系统引入到我的博客里来。Astro 可以编译成 Vanilla JS，而且有恰到好处的 Reactivity。

Hexo 的主题没一个好看的，遂弃之（这个主题本来是 Hexo 的，但是作者弃坑转了 Astro）。

# 教程

接下来才是主要内容，之前是闲聊。

## 安装

Astro 官网：[astro.build](https://astro.build)

Astro 并没有一个官方的主题系统，所以本教程只适用于 [Fuwari](https://github.com/saicaca/fuwari) 主题。如果想用其他主题需要自己去[主题列表](https://astro.build/themes)找，自己看说明。

先把仓库 Clone 下来，安装依赖：
```sh
git clone http://github.com/saicaca/fuwari.git blog
cd blog

pnpm install
pnpm add sharp
```

:::warning
一定要在第一次运行之前安装 `sharp`，否则就得重装所有依赖了。别问我怎么知道的。
:::

可以运行一下看看是不是正常：
```sh
pnpm dev
```

## 配置

配置文件位于 `src/config.ts` 中。基本上 `siteConfig` 和 `profileConfig` 需要全部更改。

主题的作者贴心地为每个配置都加了类型定义，用 VSCode 编辑非常舒服。而且 Astro 自带热重载，不用每次修改都重新生成（但得刷新）。

在改 banner 和头像的时候，如果你只是简单地进行了文件级别的图像替换，那么可能刷新浏览器并不能显示新的图片（Astro 的静态文件响应似乎并没有将文件的修改日期考虑在内，永远都是 HTTP 304）。清除浏览器缓存就解决了。

## 写博客

:::note
别急着把 `src/content/posts` 里面的东西删掉。
:::

切换到项目根目录下，运行：
```sh
pnpm new-post post-name
```

这将在 `src/content/posts` 下创建 `post-name.md` 文件。打开它，你应该能看到默认的 Frontmatter。

查看 `src/content/config.ts` 就可知这一坨 YAML 里面只有 `title` 和 `published` 是必需的。不过我强烈建议你给每一篇博客都加上 `tags` 和 `category`，这样既方便读者寻找又方便整理。

在第二条横线下面就可以开始写 Markdown 了。如果你是第一次接触 Markdown 可以看看主题自带的两篇关于 Markdown 的文章。

记得把 `src/content/spec/about.md` 也一起写了，要不然你的关于页面上还是奇怪的内容。