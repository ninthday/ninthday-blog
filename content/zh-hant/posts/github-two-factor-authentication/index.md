---
title: "開啓 Github 雙因素認證"
author: "Tuvix Shih"
slug: "github-two-factor-authentication"
featured_image: "github.png"
cover: "/images/github.png"
tags: ["github"]
date: 2019-08-05T00:10:22+08:00
draft: false
---

越來越多的詐騙、盜用帳號，讓網路使用者受到名譽或是財物上的損失，傳統單純使用帳號密碼的方式已經不夠安全，今日一個帳號走天下（ex: 使用 gmail or facebook 登入）的各種應用情境，讓雙因素認證（Two-factor authentication）或是兩階段驗證（Two-step verification）來提高帳號使用安全性顯得更為重要。

<!--more-->

# 雙因素認證

進行登入的時候，認證目前的登入對象是不是「你」是很重要的，必須不容易被他人複製，也不容易與他人重複。現在常見的認證方式：
- 一種是我們已經知道的，例如密碼或者 PIN 碼；
- 一種是我們身邊有的，例如手機或者特殊的 USB 鑰匙；
- 一種是我們與生俱來的，例如指紋、臉部特徵或者其他特徵。

雙因素認證（Two-factor authentication），主要是結合兩種不同的認證方式。例如結合帳號密碼與指紋，或是結合帳號密碼與手機。如果是使用手機，我們可以利用簡訊或是 TOTP 產生的密碼來作為另一個登入認證的因素。

# TOTP

基於時間的一次性密碼演算法（Time-based One-Time Password algorithm， TOTP）是一種根據預共用的金鑰與目前時間計算一次性密碼的演算法，結合一個私鑰與目前時間戳，使用一個密碼雜湊函式來生成一次性密碼。通常 30 秒為間隔產生新的密碼。
>參考： [wiki: Time-based One-Time Password algorithm](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm)


# 開啓 GitHub 雙因素認證

GitHub two-factor authentication（以下簡稱 2FA）主要有兩種方式：
1. 透過 **TOTP Apps**：使用 App 登入推薦使用 Authy 這個 App，可以設定備份機制不用擔心手機更換或是重置的時候造成無法登入的情形。

2. 透過 **SMS**：SMS 方式會因為手機收訊的情形而收到限制，安全性上也是較為容易被攔截的方式。

相較於 SMS 建議使用 TOTP mobile app 的方式有較高的安全性，而官方文件（[Configuring two-factor authentication](https://help.github.com/en/articles/configuring-two-factor-authentication)）中提到的三個軟體中，我覺得 **Authy** 在設定上相對簡單。我想就以 Authy 作為這次設定雙因素認證的說明。

## 1. GitHub 頁面中雙因素認證設定

1.1 登入 Github 之後在右上方點選帳號圖片，在下拉視窗中選擇 "settings"（如下圖）
![Setting](/images/2019080401.png)

1.2 在設定的左方選單中，選擇 "Security" 項目
![Security](/images/2019080402.png)

1.3 尚未設定 2FA 會出現如下圖的畫面，點擊 "**Enable two-factor authentication**"
![Enable 2FA](/images/2019080403.png)

1.4 接下來的畫面，選擇左邊的 "**Set up using an app**"
![Set up using an app](/images/2019080404.png)

1.5 接着會出現 Recovery codes 畫面，recovery codes 是讓我們在沒有 App 時可以透過出現在螢幕上的這些號碼做登入自己的帳號。避免手機有什麼狀況的時候會無法登入。
可以選擇下載（Download）或是印出來（Print）收好
![Recovery codes](/images/2019080405.png)

1.6 儲存 Recovery codes 後點擊 "Next" 進入 Scan QR code 頁面
![Scan QR code](/images/2019080406.png)

## 2. 安裝 Authy mobile app

2.1 下載 Authy mobile app

從平臺的線上軟體商店下載 app
- Google Play: https://play.google.com/store/apps/details?id=com.authy.authy
- App Store: https://itunes.apple.com/us/app/authy/id494168017

2.2 完成下載後，開啓 Authy App，依照指示輸入資料及電話，完成整個註冊程序。跟著軟體引導的步驟進行，這裏就不再多做說明。

2.3 在 Authy App 中，點選新增 Account，選擇 "Scan QR Code"
![Scan QR code](/images/2019080407.png)

2.4 將相機對準 step 1.6 的 QR Code 畫面，將 App 畫面中出現的 6 位數字，輸入至 QR Code 畫面下方的欄位。
![Six dis-digit code](/images/2019080408.png)

2.5 按下 "**Continue**" 就完成開啓 GitHub 2FA 的步驟了。
