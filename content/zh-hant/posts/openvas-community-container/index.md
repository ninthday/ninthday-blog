---
title: "Openvas Community Container"
author: "Tuvix Shih"
date: 2023-04-22T14:55:01+08:00
slug: "openvas-community-container"
featured_image: "greenbone-banner.jpg"
tags: ["linux","docker"]
categories: ["security","docker"]
draft: false
---

OpenVAS（Open Vulnerability Assessment Scanner）是一款開源的弱點掃描檢測工具，目前是由 [Greenbone](https://www.greenbone.net/) 維護，能夠持續的更新 NVT，SCAP...等弱點資料庫，對已知的漏洞及弱點進行掃描。OpenVAS 同時有 community 及 commercial 版本，community 使用的是 Greenbone Community Feed，沒有官方的技術性支援。

依據前人的經驗，OpenVAS 安裝不容易，建議如果要本機安裝，可以使用 Kali Linux 這個發行版本，已經包含於 Kali 套件庫中。如果不是使用 Kali Linux，像我的開發環境是 Ubuntu，就可以使用 docker container 的方式來安裝。

## 軟硬體需求
### 軟體版本
Greenbone Community Documentation 官方的文件支援下面這幾個 Linux 發行版本的安裝：
- Debian stable [(bullseye)](https://www.debian.org/releases/stable)
- Ubuntu 22.04 LTS
- Fedora 35 and 36
- CentOS 9 Stream

必須先安裝完 `docker-ce` 及 `docker compose`。

### 硬體需求
官方對於 `Greenbone Community Containers` 給出的執行需求建議。

最小需求:
- CPU Cores: 2
- Random-Access Memory: 4GB
- Hard Disk: 20GB free

建議需求:
- CPU Cores: 4
- Random-Access Memory: 8GB
- Hard Disk: 60GB free

## 使用 Setup and Start Script 安裝

官方 Greenbone Community Documentation [頁面](https://greenbone.github.io/docs/latest/22.4/container/index.html#setup-and-start-script) 上，很貼心的提供了 Setup and Start Script，讓使用者可以透過執行這個 `setup-and-start-greenbone-community-edition.sh` 的方式，就能夠快速的安裝。

一開始我選擇以 script 的方式進行安裝，但在下載執行後會出現如下的錯誤訊息。觀察了一下 shell script 中的內容，我認為很有可能是因為 `docker compose` 的版本不同，我使用的是 docker compose v2，v2 已經被規劃為 docker CLI 的一部分，所以發生這些錯誤。
```shell
...

Pulling Greenbone Community Containers 22.4
ERROR: The Compose file '.../GreenboneCommunityEdition/docker-compose-22.4.yml' is invalid because:
Unsupported config option for volumes: 'cert_data_vol'
Unsupported config option for services: 'vulnerability-tests'

...
```

我電腦中的 `docker` 及 `docker compose` 版本為：
```shell
$ docker --version
Docker version 23.0.3, build 3e7cbfd

$ docker compose version
Docker Compose version v2.17.2
```

最後，沒有透過 `setup-and-start-greenbone-community-edition.sh`，依照同頁文件前面的內容自行一個個步驟改變一下指令執行，還是可以順利完成安裝。

## 自行依照步驟安裝

### 1. 下載 docker-compose YAML 檔

在這[官方說明的頁面](https://greenbone.github.io/docs/latest/22.4/container/index.html#)中主要使用 `docker compose` 方式來安裝 **Greenbone Community Edition** 的 containers。這個方式主要是透過一個 YAML 設定檔，依照檔案裡面設定的服務內容，來安裝所需要的 containers。

可以直接使用下面語法，下載官方在 GitHub 準備好的 `docker-compose.yml` 檔案。
```shell
$ curl -f -L https://greenbone.github.io/docs/latest/_static/docker-compose-22.4.yml -o docker-compose.yml
```

或是複製下面 YAML 內容（版本為 22.4），存成 `docker-compose.yml` 檔案。
```yaml
services:
  vulnerability-tests:
    image: greenbone/vulnerability-tests
    environment:
      STORAGE_PATH: /var/lib/openvas/22.04/vt-data/nasl
    volumes:
      - vt_data_vol:/mnt

  notus-data:
    image: greenbone/notus-data
    volumes:
      - notus_data_vol:/mnt

  scap-data:
    image: greenbone/scap-data
    volumes:
      - scap_data_vol:/mnt

  cert-bund-data:
    image: greenbone/cert-bund-data
    volumes:
      - cert_data_vol:/mnt

  dfn-cert-data:
    image: greenbone/dfn-cert-data
    volumes:
      - cert_data_vol:/mnt
    depends_on:
      - cert-bund-data

  data-objects:
    image: greenbone/data-objects
    volumes:
      - data_objects_vol:/mnt

  report-formats:
    image: greenbone/report-formats
    volumes:
      - data_objects_vol:/mnt
    depends_on:
      - data-objects

  gpg-data:
    image: greenbone/gpg-data
    volumes:
      - gpg_data_vol:/mnt

  redis-server:
    image: greenbone/redis-server
    restart: on-failure
    volumes:
      - redis_socket_vol:/run/redis/

  pg-gvm:
    image: greenbone/pg-gvm:stable
    restart: on-failure
    volumes:
      - psql_data_vol:/var/lib/postgresql
      - psql_socket_vol:/var/run/postgresql

  gvmd:
    image: greenbone/gvmd:stable
    restart: on-failure
    volumes:
      - gvmd_data_vol:/var/lib/gvm
      - scap_data_vol:/var/lib/gvm/scap-data/
      - cert_data_vol:/var/lib/gvm/cert-data
      - data_objects_vol:/var/lib/gvm/data-objects/gvmd
      - vt_data_vol:/var/lib/openvas/plugins
      - psql_data_vol:/var/lib/postgresql
      - gvmd_socket_vol:/run/gvmd
      - ospd_openvas_socket_vol:/run/ospd
      - psql_socket_vol:/var/run/postgresql
    depends_on:
      pg-gvm:
        condition: service_started
      scap-data:
        condition: service_completed_successfully
      cert-bund-data:
        condition: service_completed_successfully
      dfn-cert-data:
        condition: service_completed_successfully
      data-objects:
        condition: service_completed_successfully
      report-formats:
        condition: service_completed_successfully

  gsa:
    image: greenbone/gsa:stable
    restart: on-failure
    ports:
      - 9392:80
    volumes:
      - gvmd_socket_vol:/run/gvmd
    depends_on:
      - gvmd

  ospd-openvas:
    image: greenbone/ospd-openvas:stable
    restart: on-failure
    init: true
    hostname: ospd-openvas.local
    cap_add:
      - NET_ADMIN # for capturing packages in promiscuous mode
      - NET_RAW # for raw sockets e.g. used for the boreas alive detection
    security_opt:
      - seccomp=unconfined
      - apparmor=unconfined
    command:
      [
        "ospd-openvas",
        "-f",
        "--config",
        "/etc/gvm/ospd-openvas.conf",
        "--mqtt-broker-address",
        "mqtt-broker",
        "--notus-feed-dir",
        "/var/lib/notus/advisories",
        "-m",
        "666"
      ]
    volumes:
      - gpg_data_vol:/etc/openvas/gnupg
      - vt_data_vol:/var/lib/openvas/plugins
      - notus_data_vol:/var/lib/notus
      - ospd_openvas_socket_vol:/run/ospd
      - redis_socket_vol:/run/redis/
    depends_on:
      redis-server:
        condition: service_started
      gpg-data:
        condition: service_completed_successfully
      vulnerability-tests:
        condition: service_completed_successfully

  mqtt-broker:
    restart: on-failure
    image: greenbone/mqtt-broker
    ports:
      - 1883:1883
    networks:
      default:
        aliases:
          - mqtt-broker
          - broker

  notus-scanner:
    restart: on-failure
    image: greenbone/notus-scanner:stable
    volumes:
      - notus_data_vol:/var/lib/notus
      - gpg_data_vol:/etc/openvas/gnupg
    environment:
      NOTUS_SCANNER_MQTT_BROKER_ADDRESS: mqtt-broker
      NOTUS_SCANNER_PRODUCTS_DIRECTORY: /var/lib/notus/products
    depends_on:
      - mqtt-broker
      - gpg-data
      - vulnerability-tests

  gvm-tools:
    image: greenbone/gvm-tools
    volumes:
      - gvmd_socket_vol:/run/gvmd
      - ospd_openvas_socket_vol:/run/ospd
    depends_on:
      - gvmd
      - ospd-openvas

volumes:
  gpg_data_vol:
  scap_data_vol:
  cert_data_vol:
  data_objects_vol:
  gvmd_data_vol:
  psql_data_vol:
  vt_data_vol:
  notus_data_vol:
  psql_socket_vol:
  gvmd_socket_vol:
  ospd_openvas_socket_vol:
  redis_socket_vol:
```

### 2. 下載及啓動 containers

透過下面的語法，直接下載 Greenbone Community Containers，下載過程會需要一下下時間。
```shell
$ docker compose -f docker-compose.yml -p greenbone-community-edition pull

[+] Running 111/16
 ✔ redis-server 4 layers [⣿⣿⣿⣿]      0B/0B      Pulled                                                            86.7s 
 ✔ ospd-openvas 21 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                         165.9s 
 ✔ gpg-data 2 layers [⣿⣿]      0B/0B      Pulled                                                                  62.3s 
 ✔ notus-data 4 layers [⣿⣿⣿⣿]      0B/0B      Pulled                                                              98.7s 
 ✔ scap-data 2 layers [⣿⣿]      0B/0B      Pulled                                                                 74.7s 
 ✔ mqtt-broker 3 layers [⣿⣿⣿]      0B/0B      Pulled                                                              92.6s 
 ✔ gvmd 10 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                            144.5s 
 ✔ dfn-cert-data 2 layers [⣿⣿]      0B/0B      Pulled                                                             43.3s 
 ✔ gvm-tools 7 layers [⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                           127.8s 
 ✔ cert-bund-data 2 layers [⣿⣿]      0B/0B      Pulled                                                            44.4s 
 ✔ vulnerability-tests 3 layers [⣿⣿⣿]      0B/0B      Pulled                                                      86.0s 
 ✔ gsa 11 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                            139.8s 
 ✔ notus-scanner 9 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                     147.7s 
 ✔ report-formats 2 layers [⣿⣿]      0B/0B      Pulled                                                            31.6s 
 ✔ pg-gvm 11 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                         158.9s 
 ✔ data-objects 2 layers [⣿⣿]      0B/0B      Pulled                                                              33.3s 
```

都出現 Pulled 下載完成後，就可以透過下面的方式進行啓動。
> Note: `-d` 以背景方式啓動

```shell
$ docker compose -f docker-compose.yml -p greenbone-community-edition up -d

[+] Running 29/29
 ✔ Network greenbone-community-edition_default                   Created                                           0.1s 
 ✔ Volume "greenbone-community-edition_gvmd_socket_vol"          Created                                           0.0s 
 ✔ Volume "greenbone-community-edition_scap_data_vol"            Created                                           0.0s 
 ✔ Volume "greenbone-community-edition_psql_data_vol"            Created                                           0.0s 
 ✔ Volume "greenbone-community-edition_redis_socket_vol"         Created                                           0.0s 
 ✔ Volume "greenbone-community-edition_data_objects_vol"         Created                                           0.0s 
 ✔ Volume "greenbone-community-edition_vt_data_vol"              Created                                           0.0s 
 ✔ Volume "greenbone-community-edition_cert_data_vol"            Created                                           0.1s 
 ✔ Volume "greenbone-community-edition_psql_socket_vol"          Created                                           0.0s 
 ✔ Volume "greenbone-community-edition_ospd_openvas_socket_vol"  Created                                           0.0s 
 ✔ Volume "greenbone-community-edition_gvmd_data_vol"            Created                                           0.0s 
 ✔ Volume "greenbone-community-edition_gpg_data_vol"             Created                                           0.0s 
 ✔ Volume "greenbone-community-edition_notus_data_vol"           Created                                           0.0s 
 ✔ Container greenbone-community-edition-mqtt-broker-1           Started                                           9.2s 
 ✔ Container greenbone-community-edition-redis-server-1          Started                                          10.5s 
 ✔ Container greenbone-community-edition-vulnerability-tests-1   Exited                                           70.4s 
 ✔ Container greenbone-community-edition-gpg-data-1              Exited                                           70.4s 
 ✔ Container greenbone-community-edition-cert-bund-data-1        Exited                                           70.3s 
 ✔ Container greenbone-community-edition-notus-data-1            Started                                          15.5s 
 ✔ Container greenbone-community-edition-pg-gvm-1                Started                                          12.7s 
 ✔ Container greenbone-community-edition-data-objects-1          Exited                                           70.4s 
 ✔ Container greenbone-community-edition-scap-data-1             Exited                                           70.4s 
 ✔ Container greenbone-community-edition-report-formats-1        Exited                                           69.7s 
 ✔ Container greenbone-community-edition-dfn-cert-data-1         Exited                                           69.7s 
 ✔ Container greenbone-community-edition-notus-scanner-1         Started                                          43.8s 
 ✔ Container greenbone-community-edition-ospd-openvas-1          Started                                          70.7s 
 ✔ Container greenbone-community-edition-gvmd-1                  Started                                          68.6s 
 ✔ Container greenbone-community-edition-gvm-tools-1             Started                                          69.7s 
 ✔ Container greenbone-community-edition-gsa-1                   Started                                          69.9s
```

### 3. 設定 Admin 密碼
如果要使用自己定義的 Admin 密碼，可以使用下面的語法來做修改。
```shell
$ docker compose -f docker-compose.yml -p greenbone-community-edition \
    exec -u gvmd gvmd gvmd --user=admin --new-password=<admin-password>
```

### 4. 開啓瀏覽器開始使用
在完成前述步驟後，就可打開瀏覽器開始使用 Greenbone Community Edition 了。下面語法可以開啓系統預設的瀏覽器。或是自行打開瀏覽器，在網址列輸入 `http://127.0.0.1:9392` 就可以看到登入畫面。
```shell
$ xdg-open "http://127.0.0.1:9392" 2>/dev/null >/dev/null &
```

## 更新內容

### 1. 更新 Containers

更新 Greenbone Community Containers 的方式，只要重新拉取 images 然後再重新啓動 container 就可以了。

```shell
$ docker compose -f docker-compose.yml -p greenbone-community-edition pull

$ docker compose -f docker-compose.yml -p greenbone-community-edition up -d
```

### 2. Greenbone Community Feed 更新
Greenbone Community Feed 的同步更新有兩個部分組成：
1. 拉取新的 data container images
2. 透過 daemon 加載到記憶體和資料庫中

Greenbone Community Feed 資料是透過幾個 container images 提供的，當這些 images 被啓動之後會自動複製到 docker volume，然後 daemon 會從 docker volume 中取得資料。

下面的語法可以下載 Greenbone Community Feed 的 data container images
```shell
$ docker compose -f docker-compose.yml -p greenbone-community-edition pull notus-data vulnerability-tests scap-data dfn-cert-data cert-bund-data report-formats data-objects
```

下面的語法啓動 container images 來複製資料至 docker volume
```shell
$ docker compose -f docker-compose.yml -p greenbone-community-edition up -d notus-data vulnerability-tests scap-data dfn-cert-data cert-bund-data report-formats data-objects
```

## 結語
使用 docker compose 方式，來安裝 Greenbone Community Edition Containers 以使用 OpenVAS 弱點掃描工具，是一個相對容易且馬上可以進行弱點掃描的方法。
網頁應用軟體的普及和方便性，身為開發者的我們，對於資安更需要多些重視。定期利用 OpenVAS 這類弱點掃描工具，來發覺系統或程式的漏洞，即時的進行修補及更正，會是我們該常駐的理念，以保障使用者的安全性。

參考資料：
1. [Greenbone Community Containers 22.4 - Greenbone Community Documentation](https://greenbone.github.io/docs/latest/22.4/container/index.html#)
2. [Workflows - Greenbone Community Documentation](https://greenbone.github.io/docs/latest/22.4/container/workflows.html)
