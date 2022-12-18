---
title: "Hugo 靜態網頁部署至 GitHub Page"
author: "Tuvix Shih"
slug: "hugo-to-github-page"
featured_image: "hugo-banner.png"
tags: ["ubuntu", "hugo", "github"]
date: 2019-06-26T23:25:47+08:00
draft: false
---

# 1. 在 ubuntu 上安裝 Hugo
在 ubuntu 上能夠使用 apt 的方式安裝，套件庫的關係版本會比較舊。建議使用 snap 的方式安裝，snap 目前也通用於各種版本的 Linux。

- Snap Package
  ```bash
  snap install hugo --channel=extended
  or
  snap install hugo
  ```

- apt install

  ```bash
  sudo apt install hugo
  ```
  
- 檢查 Hugo 安裝的版本
  ```bash
  hugo version
  ```

# 2. 初始化建立網站

## 2.1 新增網站
利用 hugo cli 的指令建立檔案結構及必要的檔案。
```bash
# blog 為站臺名稱
hugo new site blog
```

初始化網站之後，在 `bolg` 資料夾下就會產生下面的文件結構
```txt
.
├── archetypes # 放生成網頁內容的模板
├── config.toml # TOML 格式設定檔
├── content # 放 markdown 文件
├── data # 放 Hugo 處理的數據資料
├── layouts # 放佈局相關內容
├── static # 放靜態文件、圖片、CSS和JS
└── themes # 佈景主题
```

## 2.2 設定佈景主題
Hugo 佈景主題網站有提供相當多的選擇，在這裡我以 [m10c](https://themes.gohugo.io/hugo-theme-m10c/) 這個主題為例。將佈景主題複製到 theme 資料夾下。
```bash
cd blog
git clone https://github.com/vaga/hugo-theme-m10c.git themes/m10c
```
在 `config.toml` 內加上設定內容，theme 如果有說明其他設定可以一併加入。
```bash
vim config.toml
theme = "m10c"
```
> 如果選擇的佈景主題的資料中，有 `exampleSite` 資料夾，可以參考其中的 `config.toml` 設定。

## 2.3 本地端啓動網站預覽
使用下面的語法啓動 Hugo 的伺服服務，然後就可以打開瀏覽器，連結到 [http://localhost:1313](http://localhost:1313)
```bash
hugo server -t m10c
```
> `-t` 之後接的是套用的佈景主題名稱
> `-D` 可以瀏覽草稿內容

# 3. 部署至 GitHub
GitHub Pages 有兩種形態，一種是使用者（組織）、一種為專案性質。
- User/Organization Pages
> `(https://<USERNAME|ORGANIZATION>.github.io/)`
- Project Pages
> `(https://<USERNAME|ORGANIZATION>.github.io/<PROJECT>/)`

在這裏我們使用個人的 GitHub Page 也就是 `https://<USERNAME>.github.io/`的形態。
## 3.1 在 GitHub 上建立 Repository
在 GitHub 上建立兩個 repositories，一個為放置 Hugo 內容的程式庫 `ninthday-blog`，一個是 GitHub Page 使用的程式庫 `ninthday.github.io`。

## 3.2 Clone GitHub Repository
```bash
git clone git@github.com:ninthday/ninthday-blog.git
cd ninthday-blog
```
複製貼上目前的 Hugo 專案內容到 clone 下來的資料夾中，並確認能夠正確的啓動 Hugo 伺服器瀏覽本地端網頁內容。
```bash
hugo server -t m10c 
```
## 3.3 上傳 Hugo 內容至 GitHub
將 Hugo 專案的內容 push 到 GitHub 上作為備份。
```bash
git add .
git commit -m "my first commit for hugo blog"
git push origin master
```
## 3.4 部署生成內容至 GitHub Page
前面步驟中，執行 `hugo server` 沒有真的建立一個靜態網站的內容，要生成靜態網站的內容，需要執行 `hugo` 指令，執行後會在資料夾下生成 `public`資料夾，這個資料夾下的內容才是靜態的網頁內容，也就是要部署至 GitHub Page 的內容。
GitHub Page 的內容要連結部署至先前步驟建立的 `<USERNAME>.github.io` repository，在官網的建議中可以使用 `git submodule` 方式。
```bash
git submodule add -b master git@github.com:<USERNAME>/<USERNAME>.github.io.git public
```

生成靜態網站內容
```bash
hugo -t m10c
```

切換至 `public` 資料夾，Push submodule 至 GitHub 中
```bash
cd public
git push origin master
```
## 3.5 瀏覽網站
上傳過一陣子之後，可以開啓瀏覽器連結到自己的 GitHub Page，輸入 `https://<USERNAME>.github.io`

# 4. 自動部署 Script
完成上述內容，已經能夠使用 Hugo 產生靜態網站並部署至 GitHub Page。接著參考官網上的說明內容，建立一個自動部署的 shell script，讓部署的流程更為簡單。

建立 `deploy.sh` 檔案，加入以下的內容
```bash
#!/bin/bash

echo -e "Deploying updates to GitHub..."

# Build the project.
hugo -t m10c # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public
# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back up to the Project Root
cd ..
```
修改 `deploy.sh` 檔案權限為可執行狀態 `chmod +x deploy.sh`，然後執行部署 script 寫下 commit 的內容。執行後就能生成靜態網站內容並部署至 `<USERNAME>.github.io` 上。
