---
id: Obsiaian.nvim 設置
aliases: []
title: "Obsidian.nvim 設置教學"
tags:
  - 待發表
---

# Obsidian.nvim 介紹

Obsidain.nvim 是一個 Neovim 的插件，主要的功能是讓Neovim 擁有類似 Obsidian 的功能，在實際體驗當中，可以作為 Neovim 和 Obsidian 的橋樑。

## Obsidian.nvim 安裝說明

官方要求：

- Neovim >= 0.8
- nvim-lua/plenary.nvim
- Obsidian 的 vault 目錄必須在你的系統上有設好。

這裡我按照官方網站的建議安裝後有遇到 nvim-cmp 無法順利開啟的報錯，因此我的安裝過程有些許更改， 首先創建obsidian.lua :

## Obsidian.lua

```shell
~/plungs/obsidian.lau
return {
  {
    "obsidian-nvim/obsidian.nvim",
    version = "*",
    lazy = true,
    ft = "markdown",
    opts = {
      workspaces = {
        {
          name = "personal",
          path = "C:/Users/AICML/Documents/NoteSystem",
        },
      },
      completion = {
        nvim_cmp = false,
        blink = true,
      },
    },
    config = function(_, opts)
      local obsidian = require("obsidian")
      obsidian.setup(opts)

      -- 在 markdown buffer 綁定 gf
      vim.api.nvim_create_autocmd("FileType", {
        pattern = "markdown",
        callback = function()
          -- gf: 直接跳到筆記
          vim.keymap.set("n", "gf", "<cmd>ObsidianFollowLink<CR>", { buffer = true, desc = "Obsidian follow link" })

          -- gF: 在分割視窗打開筆記（可選）
          vim.keymap.set(
            "n",
            "gF",
            "<cmd>ObsidianFollowLink vsplit<CR>",
            { buffer = true, desc = "Obsidian follow link in vsplit" }
          )
        end,
      })
    end,
  },

  {
    "saghen/blink.cmp",
    version = "*",
    opts = {
      sources = {
        default = { "lsp", "path", "obsidian" },
      },
    },
  },

  {
    "nvim-treesitter/nvim-treesitter",
    build = ":TSUpdate",
    opts = {
      ensure_installed = { "markdown", "markdown_inline", "lua", "json", "vim" },
      highlight = { enable = true },
    },
  },

  {
    "neovim/nvim-lspconfig",
    config = function()
      local lspconfig = require("lspconfig")
      lspconfig.marksman.setup({})
    end,
  },

  {
    "williamboman/mason.nvim",
    build = ":MasonUpdate",
    config = function()
      require("mason").setup()
    end,
  },

  {
    "williamboman/mason-lspconfig.nvim",
    config = function()
      require("mason-lspconfig").setup({
        ensure_installed = { "marksman" },
      })
    end,
  },
}
```

再來是 cmp.lua，這裡還需要處理補全的問題

## cmp.lua

```shell
~/plungs/cmp.lua
return {
  "hrsh7th/nvim-cmp",
  dependencies = {
    "hrsh7th/cmp-nvim-lsp",
    "hrsh7th/cmp-buffer",
    "hrsh7th/cmp-path",
  },
  opts = function(_, opts)
    local cmp = require("cmp")

    -- 確保 opts.mapping 存在
    opts.mapping = opts.mapping or {}

    opts.mapping = vim.tbl_extend("force", opts.mapping, {
      -- Enter = confirm
      ["<CR>"] = cmp.mapping(function(fallback)
        if cmp.visible() and cmp.get_active_entry() then
          cmp.confirm({ select = true })
        else
          fallback()
        end
      end, { "i", "s" }),

      -- Tab = 移動候選
      ["<Tab>"] = cmp.mapping(function(fallback)
        if cmp.visible() then
          cmp.select_next_item()
        else
          fallback()
        end
      end, { "i", "s" }),

      -- Shift+Tab = 上一個候選 (常見補充)
      ["<S-Tab>"] = cmp.mapping(function(fallback)
        if cmp.visible() then
          cmp.select_prev_item()
        else
          fallback()
        end
      end, { "i", "s" }),

      -- Shift+Enter = 強制換行 (跳過補全)
      ["<S-CR>"] = cmp.mapping(function(fallback)
        fallback() -- 直接執行原本的 Enter 行為
      end, { "i", "s" }),
    })

    return opts
  end,
}
```

## 基本功能介紹

### 1. Vault & Workspace 支援

- 你可以設定多個 vault（例如 personal、work）。
- 用 opts.workspaces 指定路徑。
- 打開不同 vault 的 .md 檔時會自動切換 context。

### 2. Note 建立與管理

```shell
:ObsidianNew # 建新筆記（可以指定標題，會自動產生檔案名和 frontmatter）。
:ObsidianToday # 快速建立日記（適合 daily notes）。
:ObsidianTemplate # 插入模板（需要先設置 templates 目錄）。
```

### 3. 搜尋與連結

```shell
:ObsidianSearch <keyword> # 在 vault 內搜尋（模糊比對檔名）。
:ObsidianQuickSwitch # 快速跳轉到某個筆記。
:ObsidianFollowLink # 在 Neovim 裡跳轉到 wikilink [[]] 或普通 markdown link。
:ObsidianBacklinks # 查看目前筆記被哪些筆記連到。
```

### 4. 外部 Obsidian 整合

```shell
- ObsidianOpen # 用 Obsidian 桌面版打開當前筆記。
- :ObsidianOpen <note> # 打開指定筆記。
```

### 5. Tag / Metadata 支援

- 內建 YAML fr ntmatter 解析，可以依據 tag 搜尋或建立新筆記。

處理 轉跳問題

```lua
vim.api.nvim_create_autocmd("FileType", {
  pattern = "markdown",
  callback = function()
    -- gf: 直接跳到筆記
    vim.keymap.set(
      "n",
      "gf",
      "<cmd>ObsidianFollowLink<CR>",
      { buffer = true, desc = "Obsidian follow link" }
    )

    -- gF: 在分割視窗打開筆記（可選）
    vim.keymap.set(
      "n",
      "gF",
      "<cmd>ObsidianFollowLink vsplit<CR>",
      { buffer = true, desc = "Obsidian follow link in vsplit" }
    )
  end,
})
```
