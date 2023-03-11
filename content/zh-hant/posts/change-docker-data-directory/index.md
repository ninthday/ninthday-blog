---
title: "修改 Docker 預設資料目錄路徑"
author: "Tuvix Shih"
date: 2023-03-01T15:26:58+08:00
slug: "change-docker-data-directory"
featured_image: "cover.jpg"
tags: ["docker"]
categories: ["docker"]
draft: false
---

Docker 預設使用的路徑為 `/var/lib/docker` 這個資料夾會儲存所有的 images, volumes 等等的內容，隨着使用時間會漸漸增加到非常大。如果和我一樣 `/var` 是掛載的獨立分割區，空間被佔用滿了就會影響系統的運行。

我們可以透過修改設定的方式，將預設的資料路徑更改至不同位置。在開始之前，我們用 `docker info` 指令，先觀察 `Docker Root Dir` 目前使用的預設資料夾路徑是什麼。
```shell
$ docker info

...略...
Docker Root Dir: /var/lib/docker
Debug Mode: false
...略...
```

瞭解現在的路徑，在完成所有步驟後可以用來比對是否更動。我們可以開始進行修改的步驟如下：

### 1. 停止 docker 運行

停止正在運行的 docker daemon

```shell
$ sudo systemctl stop docker.service
$ sudo systemctl stop docker.socket 
```

確認 docker 是不是真的停止運行

```shell
$ sudo systemctl status docker.service
```

### 2.  新增設定檔

新增一個設定檔來告訴 docker 資料目錄的位置，設定檔的位置及檔案名稱為 `/etc/docker/daemon.json`， 設定檔為 json 格式內容填寫如下：

```json
{  
  "data-root": "/path/to/your/new/docker/root"  
}
```

> Note:
> `/path/to/your/new/docker/root` 為要用來放置 docker 資料目錄的新路徑

### 3. 複製當前資料目錄內容

使用 `rsync` 或是 `cp` 指令，將當前資料目錄的內容複製至新的位置。

```shell
$ sudo rsync -aP /var/lib/docker/ "/path/to/your/new/docker/root"
or 
$ sudo cp -rp /var/lib/docker/* "/path/to/your/new/docker/root/"
```

### 4. 修改舊資料目錄名稱

在完成前述複製步驟之後，將舊的資料目錄修改名稱，目的是希望用來確認 docker daemon 不會去用原來的資料目錄。（會找不到路徑）

```shell
$ sudo mv /var/lib/docker /var/lib/docker.old
```


### 5. 啓動 docker 及檢查

重新啓動執行 docker daemon。

```shell
$ sudo systemctl start docker
```

docker daemon 開始運行之後，再用 `docker info` 指令來檢查是不是套用可新設定的 `Docker Root Dir`。

```shell
$ docker info

...略...
Docker Root Dir: /path/to/your/new/docker/root
Debug Mode: false
...略...
```


### 6. 刪除舊資料夾

如果在使用上都沒有問題，就可以把原先舊的預設資料目錄移除，釋放出空間給系統使用。

```shell
$ sudo rm -rf /var/lib/docker.old
```

參考資料：
[How to change docker root data directory | by DPBD90 | Medium](https://tienbm90.medium.com/how-to-change-docker-root-data-directory-89a39be1a70b)
