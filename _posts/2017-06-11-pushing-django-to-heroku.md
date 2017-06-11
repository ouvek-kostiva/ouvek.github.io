---
layout: post
title:  "將 Django 放到 Heroku"
date:   2017-06-11 15:00:00 +0800
---

# 將 Django 作業放到 Heroku

## ◢▆▅▄▃ 崩╰(〒皿〒)╯潰 ▃▄▅▆◣

## _問題1_ Heroku 上不能用SQLite當資料庫

#### 還好 Heroku 有提供 PostgreSQL 的空間呢呢呢呢 (然後限制只能有 10000 行)

#### 所以就將資料丟到 MySQL 了 (學校的 -(￢∀￢)σ) LOAD DATA LOCAL INFILE 被禁所以只好用 INSERT, 花了好幾個小時 ಥ_ಥ

## _問題2_ Relation **Table** not found

#### 原來是 Django 沒有預設安裝 MySQL 的 Connector, 所以要 Push 到 Heroku 要加在 Requirements.txt 裡

#### 然後 Python 的 MySQL Connector 一堆裝失敗 `Push Failed` **Push Failed** Push Failed

#### 最後是 `mysqlclient` 可以用 (放在 requirements.txt)