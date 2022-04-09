---
title: "Python Django 基本指令"
date: 2022-02-08T15:17:30+08:00
draft: false
tags: 
  - Python
  - Django
description: "這裡來介紹一下python & Django幾本的指令操作。"
---

# Python & Django 基本指令

## 虛擬環境
``` cmd
# 建立虛擬環境
py -3 -m venv venv

# 進入虛擬環境
.\venv\Scripts\activate.bat

# 離開虛擬環境
.\venv\Scripts\deactivate.bat
```

## 套件管理
``` cmd
# 安裝套件
pip install <package>

# 列出有安裝的套件
pip list

# 將環境有安裝的套件匯出到 requirements.txt
pip freeze > requirements.txt

# 安裝 requirements.txt 上的套件
pip install -r requirements.txt
```

## django 專案建立
``` cmd
# 建立 django 專案 （使用第二行）
django-admin startproject <ProjectName>
py -m django startproject <ProjectName>

# 建立 app
py manage.py startapp <AppName>

# Run Develop Server
py manage.py runserver 0.0.0.0:8000
```

## 檔案結構
```
|   manage.py			# Django管理腳本，創建應用、資料庫通訊、啟動開發伺服器
|
+---Catalog				## Start App
|   |   admin.py
|   |   apps.py
|   |   models.py		### Model
|   |   tests.py
|   |   views.py		### Controller
|   |   __init__.py
|   |
|   \---migrations		# ORM 資料庫 Migrations 的地方
|           __init__.py
|
\---MySite				## Start Project
    |   asgi.py
    |   settings.py		# 網站配置，資料庫設定等等
    |   urls.py			# 網站 Router 對應，Url對應的View
    |   wsgi.py			# Django應用和網路服務器之間的通訊
    |   __init__.py
    |
```

## 資料來源
[Django 教學 2: 創建一個骨架網站](https://developer.mozilla.org/zh-TW/docs/Learn/Server-side/Django/skeleton_website)