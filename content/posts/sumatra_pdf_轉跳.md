---
id: neovim_latex_pdf
aliases: []
title: Sumtra PDF 轉跳設定
date: 2025-09-01T15:45:00+08:00
draft: false
tags:
  - LaTex
  - Sumtra
  - Neovim
---

## 前言

在 Neovim 中進行 LaTeX 編輯時，常見的工作流程是搭配 Zathura 作為 PDF 閱讀器，因其支援順暢的同步跳轉功能。然而在 Windows 環境 下並無法使用 Zathura，因此需要尋找替代方案。

Sumatra PDF 是 Windows 上輕量且相容性佳的選擇，不僅啟動速度快，也能透過額外設定支援「從 PDF 點選內容 → 跳回對應 .tex 原始碼」的功能。不過，這個功能並非安裝後就能立即使用，仍需進行一些配置。

本文將示範如何整合 Neovim 與 Sumatra，讓你能在 PDF 與 LaTeX 檔案之間順暢切換，同時避免每次跳轉時彈出惱人的命令提示視窗。

## 安裝步驟

### 1/ 安裝 neovim-remote

Neovim 本身只會啟動一個獨立的編輯器實例。如果我們單純從 Sumatra 下指令呼叫 Neovim，它只會開一個新的視窗，而不是把游標移到已經打開的檔案裡。

neovim-remote（簡稱 nvr）就是用來解決這個問題的：

它能夠把命令「傳送」給已經執行中的 Neovim 實例。

這樣當你在 PDF 點擊某一行時，Neovim 不會重開，而是直接把游標移到對應的 LaTeX 原始碼位置。

```Shell
pip install --user neovim-remote
```

### 2/ 創建 nvr-silent.vbs

如果直接用 nvr 呼叫 Neovim，Windows 會在跳轉時彈出一個 命令提示視窗 (cmd)，雖然功能正常，但非常干擾。

為了解決這個問題，可以利用 VBScript (.vbs) 來「靜默」執行命令：

.vbs 可以在背景執行程式，不會彈出黑色的 cmd 視窗。

我們在腳本裡呼叫 nvr --remote-silent，就能安靜地把跳轉命令送進 Neovim

創建 nvr-silent.vbs，並寫入下面內容

```vbs
Set WshShell = CreateObject("WScript.Shell")
WshShell.Run """" & "C:\Users\AICML\AppData\Roaming\Python\Python312\Scripts\nvr.exe" & """ --remote-silent +" & WScript.Arguments(0) & " """ & WScript.Arguments(1) & """", 0, false
```

### 3/ Sumatra 設定

進Sumatra 左上角，設定、選項、設定反相收尋命令列，輸入

```shell
"C:\Windows\System32\wscript.exe" "C:\Users\AICML\AppData\Roaming\Python\Python312\Scripts\nvr-silent.vbs" %l "%f"
```
