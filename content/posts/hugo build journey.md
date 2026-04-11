---
title: "我的 Hugo 博客搭建全记录：从踩坑到胜利的完整复盘"
date: 2026-04-12T16:00:00+08:00
tags: ["Hugo", "博客搭建", "踩坑记录", "技术分享"]
categories: ["技术"]
---

## 🎉 写在前面
终于！在经历了无数次报错、404、样式全崩、Git 冲突之后，我的个人博客终于完整跑通了！从最开始的「本地裸奔、线上404」，到现在主题正常、评论系统上线，这一路真的太坎坷了，索性把所有坑都记录下来，给同样新手的朋友做个参考，也给自己留个纪念✨

---

## 🚀 搭建背景
作为一个刚入门的前端/技术爱好者，一直想拥有一个完全属于自己的个人博客，用来记录学习、生活和技术成长。对比了很多方案后，最终选择了 **Hugo + GitHub Pages** 组合：
- 完全免费，用 GitHub 托管，不用买服务器
- 生成速度极快，静态页面访问流畅
- 主题生态丰富，高度可定制化
- 用 Markdown 写文章，简单高效

本以为是「一键部署」的轻松活，没想到直接踩满了所有新手坑…

---

## 🚨 全程踩坑记录（血泪史）
### 1.  第一坑：主题样式全崩，本地/线上全是「裸奔」页面
最开始用了 Paper 主题，结果本地打开就是纯文字、蓝色超链接，完全没有任何样式，相当于只渲染了 HTML 骨架，没穿「衣服」。
**原因**：主题配置错误 + 样式文件损坏 + 浏览器缓存三重干扰
**解决**：果断换成成熟的 Moments 主题，彻底删除旧主题，强制切换主题配置，全链路清缓存，终于样式正常。

### 2.  第二坑：Git 推送失败，远程仓库冲突
改完配置准备推送到 GitHub，结果直接报错：`Updates were rejected because the remote contains work that you do not have locally`
**原因**：在 GitHub 网页上删文件、改配置，直接修改了远程仓库，本地没同步，导致提交记录不一致
**解决**：用 `git pull origin main --rebase` 同步远程更新，再执行推送，完美解决冲突。

### 3.  第三坑：GitHub Actions 部署卡壳，本地404
线上部署成功了，结果本地直接 404 Not Found，线上正常本地崩
**原因**：首页 `_index.md` 文件缺失/格式错误，Hugo 找不到首页文件
**解决**：补全正确的首页 Front Matter，用 `hugo --gc` 强制清缓存，重新构建，404 直接消失。

### 4.  第四坑：TOML 配置语法错误，服务直接启动失败
最崩溃的坑！改配置时反复报错：`toml: key social should be a table, not a value`
**原因**：TOML 对层级、缩进、符号要求极高，手动改配置时缩进错误、符号不规范，导致解析失败
**解决**：用极简配置兜底，砍掉所有可能出错的嵌套层级，先让服务跑起来，再逐步加配置。

### 5.  第五坑：评论系统配置，从授权到生效的全流程
想给博客加评论，选了 Utterances（GitHub Issue 驱动，免费无广告），结果又踩了权限、配置、主题适配的坑
**解决**：严格按最小权限原则授权，只给博客仓库权限，用正确的 TOML 配置，最终在文章页成功加载评论区。

---

## ✅ 最终成功的完整配置（给新手抄作业）
### 1.  核心 `hugo.toml` 配置（Paper 主题+评论系统，零语法错误）
```toml
baseURL = "https://wee533.github.io/ayublog.github.io/"
languageCode = "zh-CN"
title = "我的 papers 博客"
theme = "hugo-paper-main"

relativeURLs = false
canonifyURLs = true
enableRobotsTXT = true

[params]
theme = "light"
subtitle = "欢迎来到我的 Hugo 博客 🎉"
copyright = "© 2026 我的 papers 博客"

[params.comment.utterances]
enable = true
repo = "wee533/ayublog.github.io"
issueTerm = "pathname"
theme = "github-light"
label = "blog-comment"

[menu]
[[menu.main]]
name = "首页"
url = "/"
weight = 1
[[menu.main]]
name = "文章"
url = "/posts/"
weight = 2
[[menu.main]]
name = "关于"
url = "/about/"
weight = 3