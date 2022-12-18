---
title: "修正 GCE 複製檔案至 GCS 權限錯誤"
author: "Tuvix Shih"
slug: "gce-copy-to-gcs-accessdeny"
featured_image: "google-cloud.jpg"
cover: "/images/gcp.png"
date: 2021-05-30T01:12:52+08:00
tags: ["GCP", "GCE", "GCS"]
categories: ["GCP"]
draft: false
---

## 複製出現 403 錯誤
GCE (Google Compute Engine) 在預設初始的狀態下，雖然硬碟映像檔中已經預先安裝 google cloud 工具在系統中，沒有調整權限的情形下利用 `gsutil` 工具複製檔案至 GCS (Google Cloud Storage) ，如下面語法：

```bash
$ gsutil cp /home/you/yourfile.txt gs://your-bucket
```

執行後會出現 `AccessDeniedException: 403 Insufficient Permission` 的錯誤訊息，即使在相同的專案及區域中也無法複製檔案至 GCS 上。

## 開啟 GCE Instance 的 Cloud API 存取

預設下 GCE 虛擬機器可以對同一個專案下的 GCS 具有讀取權限，如果需要寫入 GCS 就需要先進行配置調整。瀏覽器開啟 [GCP Console](https://console.cloud.google.com/compute/instances)，在下拉選單中選擇專案，在進行調整前需要先**停止**虛擬機，然後才能進行編輯。

確定 GCE 的虛擬機器停止後，點選虛擬機的名稱進入這臺虛擬機的詳細內容 (Detail)，點選上方的編輯，在編輯狀態中向下拉動畫面到`存取權範圍` (Access Scopes)，可以看到預設是選在 `允許預設存取權`，如果對於各項API熟悉，可以選擇 `針對各個 API 設定存取權` 然後在**儲存空間**部分將預設 `唯讀` 更改成 `讀寫`。

如果不想一個一個調整，可以直接選擇 `允許所有 Cloud API 的完整存取權` 這個項目。完成設定儲存後重新啟動 GCE 虛擬機器。

## 清除 gsutil 快取
完成重新啟動 GCE虛擬伺服器後，試著再執行複製檔案至 GCP 的語法。

```bash
$ gsutil cp /home/you/yourfile.txt gs://your-bucket
```

如果仍然遇到  `AccessDeniedException: 403 Insufficient Permission` 的錯誤訊息，試著清除家目錄下的 .gsutil 快取資料夾。

```bash
$ rm -r ~/.gsutil
```

再重新執行一次複製應該就可以獲得成功。

```bash
$ gsutil cp /home/you/yourfile.txt gs://your-bucket
...
Operation completed over 1 objects.
```
