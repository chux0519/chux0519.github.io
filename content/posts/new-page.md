---
title: "新的个人站"
date: 2020-04-02T14:43:09+08:00
showDate: true
draft: false
tags: ["blog","story"]
---

# 新的个人站

最近看到司徒正美去世的消息，有点惋惜又有点惆怅。惋惜的是天妒英才，惆怅的是好歹大佬离开时留下了一些好的书、一些好的框架，让人们都记得他，而这世上大多数人离开时，可能不会留有一点痕迹。同时又让人警醒，工作和生活，事业和身体，都要注意好平衡。

隔了很久，翻出了很久之前的准备用来记录博客的 repo，直到租的机器过期了，博客还是没有几篇。随着时间推移，自己越来越不愿意折腾，趁着一时兴起，今天就把博客（日记）折腾一下，搭起来，用一下，希望做到”留下点什么“。

## 解决方案

博客的生成方案，我还是采用静态网站生成器，这里使用到的是 `hugo`。网站的部署方案分为两条线，一是使用 github pages 来做静态网站，二是用自己的一台服务器做镜像。

### github pages

不了解的朋友可以看下面的参考链接，简单的说就是 github 为用户提供了便利，使用 `$username.github.io` 为名字的 repo，会自动使用 repo 内容的 master 分支作为 root，如果内容合法就可以在 `$username.github.io` 这个链接上访问得到内容。

这一次和往常不同的是，我的源代码和 hugo 最后打包出来的静态网站内容都是放在这个 repo 下的，通过 github 的 `action`，监听 `release` 分支的提交，然后触发编译，最后部署到 master 分支。整个过程显得很流畅，体验很好。后面有机会我会单独再深入讲一下如何配置 action 做到这 commit 后自动 build 和部署。

### 同步

我自己其实最近在某提供商租了一台机器，于是想要做同步。其实做法很简单，利用 nginx 起一个静态站点，然后设置一个 crontab 任务，定时地同步上面提到的 `$username.github.io` repo，rsync 内容到 nginx 的网站目录下即可。

## 参考链接

- [Working with GitHub Pages](https://help.github.com/en/github/working-with-github-pages)
- [github actions](https://github.com/features/actions)
- [peaceiris/actions-hugo](https://github.com/peaceiris/actions-hugo)
- [workflow 内容示例](https://github.com/chux0519/chux0519.github.io/blob/e7e2cf708306ee26a9b398d25a69693a6c4c5086/.github/workflows/gh-pages.yml#L1)
- [hugo](https://gohugo.io/)


