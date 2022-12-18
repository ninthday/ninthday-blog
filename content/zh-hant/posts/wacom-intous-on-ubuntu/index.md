---
title: "Ubuntu 上使用 Wacom Intuos 繪圖板"
author: "Tuvix Shih"
slug: "wacom-intous-on-ubuntu"
featured_image: "cover.png"
tags: ["ubuntu", "wacom"]
date: 2020-05-17T22:27:09+08:00
draft: false
---

基本上 Ubuntu 18.04 在接上 INTOUS 繪圖板之後，已經可以偵測到並且能夠直接使用。但是只能當作滑鼠一般使用，無法發揮繪圖板功能。必須要再安裝驅動程式來控制。

## 1. 安裝 xf86-input-wacom 驅動程式
首先在系統上安裝 `xf86-input-wacom` 驅動程式

```bash 
$ sudo apt-get install autoconf pkg-config make xutils-dev libtool xserver-xorg-dev$(dpkg -S $(which Xorg) | grep -Eo -- "-hwe-[^:]*") libx11-dev libxi-dev libxrandr-dev libxinerama-dev libudev-dev
```
安裝完成後，重新啓動電腦。

---

## 2. 設定 wacam 設備
使用 `xsetwacom` 指令列出目前的 wacom 設備
```bash
# 列出目前的設備
$ xsetwacom --list devices 
```

在我的電腦上輸出的結果：
```bash
Wacom Intuos PT S Pen stylus    	id: 12	type: STYLUS    
Wacom Intuos PT S Finger touch  	id: 13	type: TOUCH     
Wacom Intuos PT S Pad pad       	id: 14	type: PAD       
Wacom Intuos PT S Pen eraser    	id: 16	type: ERASER
```
> `Wacom Intuos PT S Pen stylus` 為繪圖筆的設定
> 繪圖板上面的按鍵，可以透過 `Wacom Intuos PT S Pad pad` 設定

---

## 3. 設定對應的螢幕
在螢幕很便宜的世代，大部分的使用者應該跟我一樣有兩個螢幕，並且設定為延伸螢幕讓整個視野可以很大。而繪圖板在預設情形下，會對應到整個延伸之後的大小。

```bash
# 取得螢幕資訊
$ xrandr -q

Screen 0: minimum 320 x 200, current 3840 x 1080, maximum 16384 x 16384
DisplayPort-0 connected primary 1920x1080+1920+0 (normal left inverted right x axis y axis) 598mm x 336mm
   1920x1080     60.00*+  74.97    50.00    59.94  
   1680x1050     59.95  
...
HDMI-A-0 connected 1920x1080+0+0 (normal left inverted right x axis y axis) 598mm x 336mm
   1920x1080     60.00*+  50.00    59.94  
   1680x1050     59.88  
... 

```

這裏我們可以看到電腦目前接着的螢幕資訊，一個是接在 `DisplayPort-0` 一個是接在 `HDMI-A-0` 上。使用 `xsetwacom` 指令，將繪圖板限定在 `DisplayPort-0` 這個螢幕。

```bash
# 以編號方式設定
$ xsetwacom --set 12 MapToOutput DisplayPort-0 

or
# 以名稱方式設定
$ xsetwacom --set "Wacom Intuos PT S Pen stylus" MapToOutput DisplayPort-0 
```

---

## 4. 設定繪圖板作業區域與螢幕比例

繪圖板作用的區域和螢幕的比例並不相同，在使用時就有可能因為比例關係，讓畫正圓變成橢圓，正方形變成長方形。

以 `xsetwacom --get` 取得繪圖板操作區域
```bash
# 取得操作區域
$ xsetwacom --get 12 area 

0 0 15200 9500
```
> 也可以使用名稱 `xsetwacom --get "Wacom Intuos PT S Pen stylus" area `

接下來就是計算繪圖作業區域與螢幕區域，然後換算成正確的高度：

```text
繪圖板比例：15200/9500 = 1.6 = 16:10
螢幕畫面比例：1920/1080 = 1.7778 = 16:9
```

繪圖板的作業區域範圍與螢幕畫面對應，也就是 `15200 x 9500(16:10)` 對應 `1920 x 1080 (16:9)`。為了能夠正常使用，我們要將繪圖板的作業區域從 16:10 限定為 16:9，讓兩個的比例相同。

我們要將大的縮小，在寬度比例相同下，繪圖板的高度比例由 10 縮小到 9，所以將繪圖板設定的作業高度調整縮小。

繪圖板的作業區域高度計算：
`高度 = (1080/1920) * 15200 = 8550`

計算完高度後，我們把新的繪圖板作業高度寫回設定。
```bash
# 設定繪圖板作業區域範圍
$ xsetwacom --set 12 area 0 0 15200 8550

```

---

## 5. 設定繪圖軟體 GIMP
繪圖板有感壓功能，在 Linux 最常用的繪圖軟體 `GIMP` 上使用時卻沒有。我們需要到軟體裡去設定使用感壓繪圖功能。

1. 開啓繪圖軟體 GIMP
2. `Edit` → `Preferences`
3. `Input Devices` → `Configure Extended Input Devices`

    選擇 device 名稱 `Wacom Intuos PT S Pen stylus`，設定至 **screen** mode
4. 儲存設定，關閉對話視窗



參考資料：
1. http://evanlab.blogspot.com/2018/12/wacom-intuos-bt-m-ctl-6100wl-ubuntu-1804.html
2. https://askubuntu.com/questions/48771/how-to-set-pressure-sensitivity-in-gimp-to-control-line-thickness
3. https://joshuawoehlke.com/wacom-intuos-and-xsetwacom-on-ubuntu-18-04/
4. https://github.com/linuxwacom/xf86-input-wacom/wiki/Building-The-Driver
