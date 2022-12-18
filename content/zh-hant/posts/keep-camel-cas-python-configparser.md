---
title: "保留 Configparser 儲存後英文字母大小寫"
author: "Tuvix Shih"
slug: "keep-camel-cas-python-configparser"
cover: "/images/python.png"
date: 2019-07-20T10:17:15+08:00
tags: ["python"]
categories: ["python"]
draft: false
---

對於程式執行階段，因為可能在不同設備或是環境有不同設定內容，使用 `.ini` INI 設定檔讀取或寫入的方式管理相當方便。利用 Python 內建的 `ConfigParser` 來處理 INI 設定檔相當方便，但在讀取後再儲存保留時，會將內容全都轉換為小寫字母。在區分字母大小寫的 Linux 系統中，希望能夠保留原來字母大小，便於閱讀和再使用。

<!--more-->

`configparser.ConfigParser()` 起源 `ConfigParser.RawConfigParser()`，在 python 說明文件中提到
> All option names are passed through the optionxform() method. Its default implementation converts option names to lower case.

因為所有的 option names 都會經過 `optionxform()` 方法，都會被轉換為小寫，導致我們使用 **write** 方式寫入保存 INI 檔的時候，Option 名稱都轉為小寫字母的原因。

我們可以透過自定 `optionxform` 屬性來達成讀取後傳回原始的字串，如下內容將其設定為 `str`，將可以留大小寫字母區分。

```python
cfgparser = ConfigParser()
cfgparser.optionxform = str
```

參考資料：

1. [configparser - Configuration file parser](https://docs.python.org/3/library/configparser.html)

2. [ConfigParser reads capital keys and make them lower case](https://stackoverflow.com/questions/19359556/configparser-reads-capital-keys-and-make-them-lower-case)
