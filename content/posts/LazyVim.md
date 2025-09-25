---
id: LazyVim_設置
aliases: []
tags: []
draft: false
title: "LazyVim 安裝與設置"
date: 2025-09-18T15:45:00+08:00
summary: "LazyVim 安裝簡易教學"
---

# 前言

為了縮短 Neovim 的安裝流程以及避免安裝過程當中的各種意外，這裡選擇使用 LazyVim(一個預配置好的Neovim 版本)，裡面包含了大部分人會用到的功能和插件。

## LazyVim 安裝

在安裝的部分完全參考[官網設置](https://www.lazyvim.org/installation)，採用最預設的安裝流程。
這裡先備份過往的 Neovim 設置(如果未曾安裝過Neovim 可以跳過):

```shell
# required
Move-Item $env:LOCALAPPDATA\nvim $env:LOCALAPPDATA\nvim.bak

# optional but recommended
Move-Item $env:LOCALAPPDATA\nvim-data $env:LOCALAPPDATA\nvim-data.bak
```

首先從git 庫安裝程式:

```shell
git clone https://github.com/LazyVim/starter $env:LOCALAPPDATA\nvim
```

接下來移除裡面的git :

```shell
Remove-Item $env:LOCALAPPDATA\nvim\.git -Recurse -Force
```

最後執行:

```shell
nvim
```

理論上就能開啟 LazyVim 的介面

## 下載對應的 Extras

Extras 是 LazyVim 官方推出的功能，針對不同用途將常見的插件整理成一個組合包，讓我們可以方便的下載和使用，以避免繁瑣的安裝問題。

由於筆記的格式是使用 Markdown 格式，我們需要在 Extras 介面找到 lang.markdown 並開啟，該 Extras 包含了基本的Markdown 功能，安裝完成後就能擁有基礎的 Markdown 編輯環境。

當下載完成後可以在 lazyvim.json 觀察到新增:

```json
// File: $env:LOCALAPPDATA\nvim\lazyvim.json
{
  "extras": [
    "lazyvim.plugins.extras.lang.markdown"
  ],
  "news": {
    "NEWS.md": "5202"
  },
  "version": 5
}
```

### lang.markdown 介紹

下面簡單介紹 lang.markdown 內含的插件組成和用法

#### markdown-preview.nvim (即時預覽器)

功能： 這就是讓你能在瀏覽器中看到即時渲染效果的插件。
如何使用：

```vim
:MarkdownPreview：啟動或開關預覽。
:MarkdownPreviewStop：停止預覽。
```

#### headlines.nvim (美化編輯器介面)

功能： 這個插件讓你在 Neovim 編輯器內部的 Markdown 檔案看起來更乾淨、更有結構感。它會自動隱藏標題的 # 符號，用更美觀的圖示替換，還能加上漂亮的分隔線。

### markdown Extra 設定方法

當想要修改 markdown 相關的設定時，可以在 `$env:LOCALAPPDATA\nvim\lua\plugins` 路徑下創建一個 `markdown.lua` 檔案。
系統偵測到這個檔案時，裡面的設定會覆蓋掉原本 Extra 的預設設定。

```shell
$env:LOCALAPPDATA\nvim
├── lua
│   ├── config
│   │   ├── autocmds.lua
│   │   ├── keymaps.lua
│   │   ├── lazy.lua
│   │   └── options.lua
│   └── plugins
│       ├── spec1.lua
│       ├── **
│       └── markdown.lua
└── init.lua
```

下面是 `markdown.lua` 的設定範例，你可以在[官網](https://www.lazyvim.org/extras/lang/markdown)找到更多的設定選項。

```lua
-- 路徑: $env:LOCALAPPDATA\nvim\lua\plugins\markdown.lua
-- 功能: 客製化 lang.markdown extra 中包含的插件設定

return {
  -- 覆寫 "iamcco/markdown-preview.nvim" 的預設設定
  {
    "iamcco/markdown-preview.nvim",
    opts = {
      --當離開 Markdown 文件時，自動關閉預覽視窗
      mkdp_auto_close = true,
    },
  },

  -- 如果需要，你可以在這裡加入對其他 markdown 相關插件的設定
  -- 例如:
  -- {
  --   "headlines.nvim",
  --   opts = { ... }
  -- }
}
```
