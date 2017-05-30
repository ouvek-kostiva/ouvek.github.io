---
layout: post
title:  "十年每日價量資料"
date:   2017-05-30 12:00:00 +0800
---

# 十年每日價量資料

#### [檔案](https://www.dropbox.com/sh/u2p603dijgychy5/AABh8zkOcqFvnqUumJr3t8zza?dl=0)

> 下載後在 GoogleAPIv3 資料夾執行

```python
import sqlite3
dbName = "PriceVolData.db"
tableName = "twse"
conn = sqlite3.connect(dbName)
c = conn.cursor()

out = conn.execute("SELECT indexColumn, code, date FROM twse")

dalist = []
for indexColumn, code, date in out:
    dalist.append([indexColumn,code,date])
    
conn.close()

print(dalist[10000])
```

##### 結果
> [10001, '1108', '2008-03-20 13:30:00']

> 資料庫結構為
>
> 資料表名稱: twse
>
> 欄位名稱: indexColumn 主鍵, code 股票代號, date 日期, close 收盤價, low 最低價, open 開盤價, volume 交易量

##### 從資料庫取得資料 SELECT
##### indexColumn, code, date 可放欄位名稱
##### 在 Query後 用 WHERE 選擇單日或單股資料等條件

```python
"SELECT indexColumn, code, date FROM twse"
```

##### 顯示資料
##### 同樣的在 for 與 in 之間為欄位名稱

```python
dalist = []
for indexColumn, code, date in out:
    dalist.append([indexColumn,code,date])
```

##### 用 print(dalist\[行數\]\[欄位數\]) 顯示資料
```python
print(dalist[10000])
> [10001, '1108', '2008-03-20 13:30:00']
```
