---
title: "Python 套件相依性管理工具 Poetry"
author: "Tuvix Shih"
slug: "python-poetry"
cover: "/images/python.png"
date: 2020-06-21T16:36:43+08:00
tags: ["python", "poetry"]
categories: ["python"]
draft: false
---

Python 在開發時常使用 venv 及 pip 當作開發時的環境管理，後來網路上發現其他開發者推薦的撒尿牛丸 [Pipenv](https://pypi.org/project/pipenv/)，同時處理套件相依性及虛擬環境。

後來因為套件鎖定仍會更新、更新 LOCK 速度太慢、其他開發者的貢獻沒有併入 master 等問題，造成社群中不少的聲音。社群中於是有了這樣的聲音

> Pipenv 描繪了一個美夢，讓我們為 Python 也有了其他語言一樣的套件管理方式，不過卻在後來的 Poetry 的到了更好的實踐。

讓我想嘗試看看 [Poetry](https://python-poetry.org/)，Poetry 和 Pipenv 雷同能夠做虛擬環境及套件依賴的管理，除此之外，也提供了套件打包和發佈管理的功能。

- 官方網站：https://python-poetry.org/
- GitHub：https://github.com/python-poetry/poetry
- Document：https://python-poetry.org/docs/

---

## 1. 安裝 Peotry

官方推薦的方法
```bash
$ curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python3
```
官方建議不要使用 `pip` 的方式安裝，因為有可能會在全域環境中安裝 Poetry 的相依套件，造成套件污染的情形。

```
$ poetry --version 

Poetry version 1.0.9
```
> Note:
>
> 如果出現下面錯誤，可以修改 `~/.poetry/bin/poetry`，將 `/usr/bin/env python` 修改為 `/usr/bin/env python3` 就可以修復。希望之後的 patch 能夠修復。
>```
> /home/tuvix/.poetry/lib/poetry/_vendor/py2.7/subprocess32.py:149: RuntimeWarning: The _posixsubprocess module is not being used. Child process reliability may suffer if your program uses threads.
> "program uses threads.", RuntimeWarning)
>```
> 
> 參考：
> [issue#1257-Problem vendoring subprocess32 in Ubuntu](https://github.com/python-poetry/poetry/issues/1257)

## 2. 環境設定

Poetry 在 Linux 系統的安裝路徑為 `$HOME/.poetry/bin`，需要把路徑加入到 `PATH` 環境變數中。若要立即套用至目前的 shell session，可以使用 `source $HOME/.poetry/env` 指令。

建議將上述路徑加到 bashrc 或是 zshrc 中，在開啓 terminal 的時候，就直接套用至 PATH 中。
```
export PATH=$PATH:$HOME/.poetry/bin
```

zplug 套件

[zsh-poetry](https://github.com/darvid/zsh-poetry) 這個套件，檢查資料夾中是否存在合法的 `pyproject.toml`，在切換至資料夾時自動啓動虛擬環境（virtual environment）

```bash
$ vim ~/.zshrc

# python-poetry
zplug "darvid/zsh-poetry"
```
## 3. 開始 Poetry

### 3.1 專案設定
#### 現有專案
在已經存在的專案中可以使用 `poetry init` 初始化 poetry，跟著提示輸入詢問的回應，不確定的回應可以直接 `Enter` 使用預設內容，之後都可以再編輯，完成後便會在專案中建立 `pyproject.toml` 檔案。

```bash
# 現有專案
$ poetry init
```

#### 創建新專案
如果要透過 poetry 創建一個新的 python 專案，使用 `poetry new <專案名稱>` 即可建立一個專案模板的資料夾結構。如果建立的時候需要 `src` 資料夾，可以加上 `--src` 選項。套件名稱弱想與資料架不同，可以使用 `--name` 指定。
```bash
# 建立新專案 poetry-demo
$ poetry new poetry-demo

# 建立含 src 的新專案
$ poetry new  --src poetry-demo

# 建立新專案時專案名稱和資料夾不同
$ poetry new my-folder --name poetry-demo
```

在專案建立後，poetry 安排的資料夾目錄結構。
```bash
# 目錄結構
poetry-demo
├── pyproject.toml
├── README.rst
├── poetry_demo
│   └── __init__.py
└── tests
    ├── __init__.py
    └── test_poetry_demo.py
```

Python 在 PEP 518 引入了新的標準，使用 `pyproject.toml` 檔案來管理各項 meta 資訊，peotry 使用 `pyproject.toml` 作為配置的設定檔，專案建立後的 `pyproject.toml` 初始內容如下：
```toml
# 專案基本資料
[tool.poetry]
name = "poetry-demo"
version = "0.1.0"
description = "測試"
authors = ["Tuvix Shih <tuvix@ninthday.info>"]

# 項目依賴
[tool.poetry.dependencies]
python = "*"

# 開發時的依賴
[tool.poetry.dev-dependencies]
pytest = "^3.6"
```

初始設定的內容為基礎資訊內容，提供的資訊有限，更完整的 `pyproject.toml` 內容會像：
```ini
# 專案基本資料
[tool.poetry]
name = "poetry-demo"
version = "0.1.0"
description = "測試"
authors = ["Tuvix Shih <tuvix@ninthday.info>"]
maintainers = ["Tuvix Shih <tuvix@ninthday.info>"]
license = "MIT"
readme = "README.md"
homepage = ""
repository = "https://github.com/ninthday/poetry-demo/"
documentation = "https://example.com/python-poetry/"
keywords = ["Poetry","Virtual Environments","Tutorial","Packages"]

# 項目依賴
[tool.poetry.dependencies]
python = "*"
loguru = "*"

# 開發時的依賴
[tool.poetry.dev-dependencies]
pytest = "^3.6"

[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"

# 選擇性，對專案使用者有用的連結
[tool.poetry.urls]
issues = "https://github.com/ninthday/poetry-demo/issues"
```

### 3.2 建立及啓動虛擬環境
確認資料夾中有 `pyproject.toml` 檔案，使用 `poetry install` 建立虛擬環境，這個指令會讀取 `pyproject.toml` 裡的依賴並且安裝套件。不想安裝開發時才會用到的依賴，可以加上 `--no-dev`。
```bash
# 建立虛擬環境並安裝套件
$ poetry install

# 建立虛擬環境不安裝開發時的依賴項目
$ poetry install --no-dev
```

### 3.3 虛擬環境執行
想要即時的在虛擬環境中執行指令，可以使用 `poetry run <執行的指令>` 的方式。如果要進入虛擬環境則可以用 `poetry shell`。
```bash
# 即時的在虛擬環境中執行指令
$ poetry run python3 demo.py

# 進入啓動的虛擬環境
$ poetry shell
```

## 4. 套件管理
Python 條件管理，`poetry` 相較於 `pip` 更為符合人性也做得更好，對於套件相依性在升級或是移除後的處理都做的更好，也更為精簡。

### 4.1 安裝套件
poetry 可以透過 `poetry add <套件名稱>` 指令來安裝套件，如果有些套件是開發時才需要，例如 pytest ，可以在安裝時加入 --dev 參數，將該套件標記為 dev 環境才需要安裝。
```bash
# 安裝套件
$ poetry add requests

# 安裝開發時的依賴套件
$ poetry add pytest --dev

# 安裝特定版本
$ poetry add pyspark=2.4.1
```
### 4.2 追蹤套件
使用 `poetry show` 可以查看目前所有安裝套件的依賴，加上 `--tree` 可以以樹狀圖的方式查看套件依賴。`--no-dev` 不顯示開發時的依賴。`--latest (-l)` 顯示目前版本及套件最後版本。`--outdated` 僅顯示有新版本的依賴套件。
```bash
# 查看安裝的套件
$ poetry show 

# 樹狀圖查看套件的依賴關係
$ poetry show --tree

# 查詢時不顯示開發環境版本
$ poetry show --no-dev

# 查詢套件版本及最後版本
$ poetry show --latest

# 查詢有新版本的依賴
$ poetry show --outdated
```
如果想要查詢特定套件的詳細資訊，可以在指令後接套件名稱 `poetry show <套件名稱>`。
```bash
# 查詢特定套件
$ poetry show requests

name         : requests
version      : 2.24.0
description  : Python HTTP for Humans.

dependencies
 - certifi >=2017.4.17
 - chardet >=3.0.2,<4
 - idna >=2.5,<3
 - urllib3 >=1.21.1,<1.25.0 || >1.25.0,<1.25.1 || >1.25.1,<1.26
```

### 4.3 更新套件
獲得最新套件的版本及依賴關係，並更新 `poetry.lock` 檔案，使用 `poetry update` 指令。如果要更新特定套件而不是全部，在上述指令後接着套件名稱 `poetry update <套件名稱>`。
其他參數，`--dry-run` 輸出操作過程但沒有真的執行。
```bash
# 更新專案中所有套件及依賴
$ poetry update

# 更新特定套件
$ poetry update requests

# 更新時不包含開發環境版本
$ poetry update --no-dev

# 輸出更新操作過程但不執行
$ poetry update --dry-run
```

### 4.4 移除套件
使用 `poetry remove <套件名稱>` 來移除指定的套件。
```bash
# 移除套件
poetry remove requests
```

## 5. 建立套件及發佈
### 5.1 建立套件
使用 `poetry build` 建立 `source` 和 `wheels` 檔案，可加上 `--format (-f)` 建立 `wheel` 或是 `sdist`。
```bash
# 建立套件
$ poetry build
```

### 5.2 發佈套件
`publish` 指令將會把 `build` 指令建立的套件發佈到遠程的儲存庫，如果是第一次發佈上傳前將會自動的註冊套件。
```bash
# 發佈套件到遠端
$ poetry publish

# 發不到指定的遠端
$ poetry publish --repository my-repository
```

## 6. Poetry 指令說明

`poetry` 提供了相當多的功能，都可以使用指令來控制
```
# poetry v.1.0.9
about          -- 關於 Poetry 的資訊
add            -- 在 pyproject.toml 中新增一個新的依賴
build          -- 建立一個套件，預設為生成 tarball 和 wheel
cache          -- 互動方式處理 poetry 暫存
check          -- 驗證 pyproject.tml 檔案的有效性
config         -- 設定或是取得 poetry 配置項目
debug          -- Debug poetry 變數
env            -- 操作/取得 poetry 的虛擬環境
export         -- 匯出 lock 檔案
help           -- 顯示指令的說明
init           -- 在目前的資料夾中建立一個基礎的 pyproject.toml 檔案
install        -- 安裝依賴項目
lock           -- 鎖定依賴項目
new            -- 建立一個新的 Python 專案
publish        -- 將套件發佈到遠端的程式庫
remove         -- 移除依賴項目
run            -- 在適當的環境中運行指令
search         -- 在遠端程式庫中搜尋套件
self           -- 操作 poetry 本身
shell          -- 進入虛擬環境的 shell
show           -- 顯示套件資訊
update         -- 依照 pyproject.toml 檔案更新依賴項目
version        -- 顯示 poetry 版本資訊
```

## 7. 升级 poetry 
使用 `self update` 指令可以將 poetry 升級到最後的穩定版本，如果想嘗試看看較新的預覽版本（可能不穩定）可以加上 `--preview` 選項。
```bash
# 升級至最後穩定版本
$ poetry self update

# 升級至預覽版本
$ poetry self update --preview
```
## 8. 匯出 requirements.txt
有時候佈署運行的環境沒有提供 poetry，也可以匯出 `requirements.txt` 的方式透過 `pip` 安裝套件。匯出的指令為 `poetry export`，目前只有提供 `requirements.txt` 的格式。
```bash
# 匯出 requirements.txt
$ poetry export -f requirements.txt -o requirements.txt

# 匯出包含開發環境會使用的內容
$ poetry export --dev -f requirements.txt -o requirements.txt
```

---

參考資料：
1. [Poetry Document](https://python-poetry.org/docs/)
2. https://myapollo.com.tw/zh-tw/python-poerty/
3. https://zhuanlan.zhihu.com/p/81025311.
4. https://shazi.info/pip-pipenv-%E5%92%8C-poetry-%E7%9A%84%E9%81%B8%E6%93%87/
5. https://www.escapelife.site/posts/fc616494.html
6. https://myoceane.fr/index.php/python-poetry/
7. https://hackersandslackers.com/python-poetry-package-manager/
