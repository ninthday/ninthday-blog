---
title: "用 Git Submodule 管理 Hugo Theme"
author: "Tuvix Shih"
slug: "git-submodule-hugo-theme"
cover: "/images/hugo.png"
date: 2019-07-12T23:20:57+08:00
tags: ["hugo", "git"]
draft: false
---

通常在 Hugo 目錄下會有一個 `theme` 資料夾，用來放置使用的佈景主題。然而每個佈景主題通常都會有自己的 GitHub Repository，而我們的 Hugo 內容也會有自己的 Repository，在內容管理上容易相互干擾，佈景主題及我們自己的 Repo 都仍然會持續更新。為了在佈景主題更新時能夠更方便的同步，和我們自己的 Repo 間的同步能夠更好的協作，可以使用 git submodule 方式加入 Hugo Theme。

<!--more-->

# Git Submodule
什麼是 git submodule？時常我們一個專案上工作時，有可能會使用到另一個 Repo 的內容，或許是一個函式庫或許是另一個套件工具，這些我們專案使用到的外部資源，都有可能是在自己的 Repo 上持續的進行中。
如果把外部的資源複製到自己的專案下，每次在外部程式碼有更新的時候，都會需要一再重複的複製，管理上變得困難。

Git 能夠通過 submodule 處理這樣問題。submodule 允許我們將一個 Git repo 當作另外一個 Git repo 的子目錄。這允許我們 clone 另外一個 repository 到自己的專案中，並且仍然保持我們的提交相對獨立。

Git submodule 加入語法雷同於 clone
```bash
$ git submodule add <repository> [<path>]
```

# Hugo theme 以 git submodule 方式加入

將 Hugo theme 以 **git submodule** 方式加入，我們同樣以 m10c 作為說明範例。

## 新增 submodule
在 Hugo 的根目錄下，輸入下列指令
```bash
$ git submodule add https://github.com/vaga/hugo-theme-m10c.git themes/m10c
```
當指令完成之後，於根目錄下會生成一個 `.gitmodules` 設定檔，裡面記錄了 git submodule 的一些設定訊息。將這個檔案一起加入到自己的 repo 中，就能夠方便管理。`.gitmodules` 內容如下

```ini
[submodule "themes/m10c"]
	path = themes/m10c
	url = https://github.com/vaga/hugo-theme-m10c.git
```
> 如果以 submodule 加入多個 theme，`.gitmodules` 中就會有多條相關的設定內容。

## 更新 submodule

如果專案下的 submodule 不多，可以進入個別的 submodule 資料夾中，執行 `git pull` 的方式去拉下更新的內容。若一次要全部一起更新所有 submodule，可以使用 `foreach` 
```bash
$ git submodule foreach --recursive git pull --rebase origin master
```

這個方式將會遞迴進入每個 submodule 中執行 `git pull --rebase origin master` 指令 。另一個方式直接使用 `--remote` 的方式一次更新。
```bash
$ git submodule update --remote --rebase
```

## 刪除 submodule
Submodule 的刪除比加入麻煩。沒有 rm 的指令，必須要手動刪除。

1. 要先進入 `.gitmodules` 中將不要的 submodule 區段刪除。
2. 用 `git rm –cached` 方式
3. 刪除 submodule 資料夾
4. 修改 `.git/config` 刪除不要的 submodule

```bash
# .gitmodules 刪除 submodule 區段
$ vim .gitmodules

$ git rm –cached themes/m10c
$ rm -rf themes/m10c

# .git/config 刪除 submodule
$ vim .git/config

# 提交到 GitHub
$ git add .
$ git commit -m "remove submodule"
$ git push origin master
```

# Clone Hugo 專案 submodule 為空？
若像我有在不同電腦進行 Hugo Blog 內文撰寫（就是懶得把電腦帶來帶去的），在另一臺用 `git clone` 下載自己的 Hugo 專案時，會發現 submodule 對應的資料夾雖然存在，但是 submodule 資料夾都是空的。`git clone`指令並不會一起下載 submodule 中的內容。

可以使用下面的指令可以將 `.gitmodules` 中的內容寫入 `.git/config`，然後再更新下載 submodule 內容。

```bash
$ git submodule init
$ git submodule update --recursive

or
# 或是合併成一行
$ git submodule update --init --recursive
```
> - `git submodule init`：根據 `.gitmodules` 的名稱和設定將資訊註冊到 `.git/config` 內。（但 `.gitmodules` 內移除的 submodule 並不會自動刪除 `.git/config` 的相關內容，需手動刪除）
> - `git submodule update`：根據已註冊 `.git/config` 的 submodule 進行更新，執行這個指令前最好可以加上 `--init`。

當然我們自己知道專案中已經包含 submodule 的時候，就可以在 clone 的時候一次一起更新下來。
```bash
$ git clone --recurse-submodules git@github.com:ninthday/ninthday-blog.git
```
---

參考資料：
1. [Git 工具 - 子模組 (Submodules)](https://git-scm.com/book/zh-tw/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E7%B5%84-Submodules)
2. [git submodule的使用](https://blog.csdn.net/wangjia55/article/details/24400501)
3. [Git Submodule 用法筆記](https://blog.chh.tw/posts/git-submodule/)
4. [Git submodule is returning blank?](https://stackoverflow.com/questions/11420701/git-submodule-is-returning-blank)
