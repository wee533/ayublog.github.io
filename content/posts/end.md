---
title: "我的 Hugo 博客搭建实录 - 终案"
date: 2026-04-15 18:00:00
lastmod: 2026-04-15 18:00:00
draft: false
tags: ["Hugo", "博客搭建", "Paper主题", "踩坑记录", "技术分享"]
categories: ["技术"]
author: "啊钰"
description: "一篇搞定 Hugo + Paper 主题博客搭建，从踩坑到完美上线的终极教程。"
---

## 一、开篇：为什么选 Hugo？

作为一个折腾党，选博客框架时，我直接把 Hexo、Jekyll、Hugo 全对比了一遍：

- **Hexo**：生态好，但 Node 版本坑多，本地构建慢，大项目容易崩
- **Jekyll**：Ruby 环境折腾，国内访问慢，对新手不友好
- **Hugo**：Go 语言写的，构建速度秒开，主题生态丰富，Paper 主题的极简风格直接戳中我，零犹豫选它。

本以为是「5 分钟搭好博客」的快乐，结果一上手，直接掉进了连环坑。

---

## 二、踩坑实录：那些让我崩溃的报错

### 坑 1：public 文件夹死活推不上 GitHub

最开始的问题，直接给我整懵了：本地 hugo 生成的 public 文件夹，里面 CSS、JS 全齐，一推 GitHub，就提示「nothing to commit」，死活传不上去。

**根源**：
Hugo 项目默认的 `.gitignore` 里，直接写了 `/public`，Git 直接把这个文件夹屏蔽了，你怎么 add 都没用。

**解决**：
1. 打开项目根目录的 `.gitignore`，把 `/public` 这一行删掉（或者加 `#` 注释掉）
2. 用强制添加命令，无视 `.gitignore` 提交：

```bash
git add -f public
git commit -m "feat: 上传完整public文件夹"
坑 2：Git 推送超时，443 端口连不上
好不容易解决了提交，一 git push，直接红屏报错：
plaintext
fatal: unable to access 'https://github.com/xxx/xxx.github.io.git/': Failed to connect to github.com port 443 after 21000 ms: Timed out
翻译成人话：网络超时，连不上 GitHub 服务器。
这是国内搭 GitHub Pages 最常见的坑。
解决方案（按优先级排序）：
最快见效：连手机热点，直接绕开网络限制，90% 的情况一次成功
代理配置：给 Git 配置代理（默认 7890）
bash
运行
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
一劳永逸：换 SSH 协议
终极兜底：GitHub 网页直接上传文件
坑 3：CSS 全裸奔！页面变成浏览器默认样式
这是最崩溃的一个坑：public 推上去了，打开博客，直接变成了「裸 HTML」—— 紫色下划线链接、无样式标签、排版全乱。
排查结论：浏览器根本没拿到 CSS 文件，HTML 里的 <link> 直接消失了。
根源：Paper 主题用了 Hugo Pipes 资源打包，要求 CSS 文件必须放在 /assets 文件夹，而我把 CSS 放在了 /static 文件夹里！
Hugo 找不到文件，直接不渲染 CSS 代码，连报错都没有。
原主题错误代码：
go
运行
{{- $main := resources.Get "main.css" | minify -}}
{{- $custom := resources.Get "custom.css" | minify -}}
{{- $css := slice $main $custom | resources.Concat "main.css" | minify | fingerprint -}}
<link rel="stylesheet" href="{{ $css.Permalink }}" integrity="{{ $css.Data.Integrity }}" />
坑 4：Hugo Pipes 抽风，CSS 死活不生成
就算把 CSS 放到 assets/，有时候还是会抽风：本地正常，一推 GitHub 样式又没了。
根源就是环境不一致。
三、终极解决方案：100% 生效的 CSS 修复
折腾了整整一天，我终于找到了不管什么环境、100% 生效的 CSS 加载方案。
第一步：彻底替换 head.html 里的 CSS 代码
打开主题的 layouts/partials/head.html，把原来的代码替换成：
html
预览
<!-- 100% 生效的静态CSS加载，彻底绕开Hugo Pipes坑 -->
<link rel="stylesheet" href="{{ `main.css` | absURL }}" />
<link rel="stylesheet" href="{{ `custom.css` | absURL }}" />
第二步：把 CSS 放回 static 文件夹
确保路径：
plaintext
项目根目录/
├── static/
│   ├── main.css
│   └── custom.css
第三步：本地重新构建 + 强制推送
bash
运行
# 彻底删除旧的public，清除缓存
hugo --gc --minify --cleanDestinationDir

# 强制提交public
git add -f public
git commit -m "fix: 彻底修复CSS加载，恢复主题样式"

# 强制推送到GitHub
git push -f origin main
第四步：强制刷新浏览器
打开博客，按：
Ctrl + Shift + R（Windows）
Cmd + Shift + R（Mac）
样式立刻恢复正常！
四、最终效果：完美运行的博客
经过这一系列折腾，我的 Hugo 博客终于完美上线了：
Paper 主题的米白背景、圆角标签、极简排版，完全正常
导航栏、文章列表、标签页，样式丝滑
本地、GitHub Pages 访问，CSS 100% 加载，再也不裸奔
从最开始的连环报错，到最终的完美运行，这种从 0 到 1 搞定一个项目的成就感，真的太爽了。
五、后续优化：让博客更丝滑
1. 开启 GitHub Actions 自动部署
彻底告别手动推 public，以后改完代码，只需要 git push，GitHub 自动帮你构建、部署，零操作。
完整配置文件 .github/workflows/deploy.yml：
yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --gc --minify --cleanDestinationDir

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
2. 接入 Cloudflare CDN
给博客加速、加防护，国内访问秒开，还能免费申请 SSL 证书，让你的网址显示「绿色小锁」。
3. 接入 Umami 访客统计
替代 Google Analytics，国内可访问，隐私友好，零广告，完美适配 Hugo 博客。
六、写在最后
折腾 Hugo 博客的这几天，真的是「一边崩溃，一边自愈」。
从最开始的「怎么推都推不上」，到「CSS 裸奔到怀疑人生」，再到最终的完美上线，每一个坑，都是成长的印记。
如果你也在折腾 Hugo 博客，遇到了和我一样的问题，照着这篇文章走，绝对能少走 99% 的弯路。
愿每一个折腾博客的人，都能拥有一个属于自己的、完美的小天地。
plaintext

---

# 🚀 现在直接推送到 GitHub（复制这 4 行）

```powershell
hugo --gc --minify
git add .
git commit -m "发布：我的Hugo博客搭建实录-终案"
git push -f origin main
推完 强制刷新 就能看到啦！🎉