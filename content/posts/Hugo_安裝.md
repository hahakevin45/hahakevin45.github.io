---
id: Hugo_安裝
aliases: []
date: 2025-08-29T15:45:00+08:00
title: Hugo + GitHub Pages + 自動部署
draft: false
tags: []
---

先備知識:

1. git
2. GitHub

## Hugo 安裝

使用 scoop 下載 huge

```shell
scoop install hugo
```

創建 hugo site

```shell
hugo new site my-notes
cd my-notes # 進入my-note 資料夾
```

## PaperMod 主題安裝

這是我選用的主題，如果要用不同的主題也是可以

從git 下載 PaperMod 主題

```shell
git init
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

## Huge 測試

新增測試用文章至my-notes/posts/，這是不能省略的一步。

```shell
hugo new posts/first-post.md
```

開啟本地預覽，這會在網址 (<http://localhost:1313/>) 產生網站。

```shell
hugo server -D
```

## GitHub 自動部署

接下來要部署到 GitHub Pages

首先在 GitHub 建一個新 repo，名字就叫 USERNAME.github.io。

接下來要在 repo 的設定當中注意

- Setting/Action/General/Workflow permissions
  - 需要選 Read and write permissions
- Setting/Environments/github-pages
  - Deployment branches and tags 應該要是空白的
- Setting/Pages/Build and Deployment
  - Source 需要選 Github Action

接下來找到 hugo.toml 進行設置，這是 hugo 主要進行設置的地方，我加入了檔案、收尋、標籤功能，還有最上面的介紹和社交軟體連結，這邊都能按照官網進行客製化。

```toml
baseURL = "https://USERNAME.github.io/"
languageCode = "zh-tw"
title = "your title"
theme = "PaperMod"
contentDir = "content"

# 啟用搜尋功能所需的設定
[outputs]
  home = ["HTML", "RSS", "JSON"]

# --- 主要參數設定 ---
[params]
  defaultTheme = "auto"
  ShowReadingTime = true
  ShowPostNavLinks = true
  enableCodeCopy = true

# --- 主頁歡迎詞 (官方推薦作法) ---
# 這會啟用 Home-Info 模式，並在主頁頂部顯示標題和內容
[params.homeInfoParams]
  Title = "Home title"
  Content = "Home Content"

# --- 主頁頂部的社交媒體圖示 ---
[[params.socialIcons]]
  name = "github"
  url = "https://github.com/USERNAME"

[[params.socialIcons]]
  name = "instagram"
  url = "https://www.instagram.com/USERNAME/"

# --- 頂部導覽列 ---
[[menu.main]]
  identifier = "archives"
  name = "檔案"
  url = "/archives/"
  weight = 10

[[menu.main]]
  identifier = "search"
  name = "搜尋"
  url = "/search/"
  weight = 20

[[menu.main]]
  identifier = "tags"
  name = "標籤"
  url = "/tags/"
  weight = 30

[markup.goldmark.renderer]
  unsafe = true
```

再來找到.github/Workflow/deplay.yml，如果沒有的話自己創建一個，並進行如下設置，這會讓github 的action 知道要如何自動部署網站。

```yml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]   # 或你主要寫作的分支
  workflow_dispatch:      # 允許手動觸發

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: true   # 如果你的 Hugo theme 是 submodule 要加這個
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "latest"
          extended: true

      - name: Build
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v4
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
```

接下來為了避免不必要的檔案上傳，在 .gitignore 添加

```gitignore
# Hugo build output
/public
/resources
.hugo_build.lock

# 系統檔案
.DS_Store
Thumbs.db
```

最後進行推送

```shell
git add .
git commit -m "Initialize Hugo site with PaperMod"
git push -u origin main
```

應該就能在網址 <https://USERNAME.github.io/> 看到網站了，如果沒有看到網站的話，建議到 Github 的 Action 介面看看運作的狀況。

## 重新建立 Hugo 

如果要在重新建立 Hugo 環境，因為 PaperMod 是 submodule，如果只用 git clone，預設不會抓子模組，所以我們需要加上指令

```shell
git submodule update --init --recursive
```

才能正常編譯。
