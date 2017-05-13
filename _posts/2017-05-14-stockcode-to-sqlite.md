---
layout: post
title:  "股票代號匯入 SQLite3"
date:   2017-05-15 01:25:00 +0800
---

# 將「上市上櫃」及「終止上市公司代碼」匯入資料庫

#### [Pandas DataFrame Pickle 檔](https://github.com/ouvek-kostiva/stockchooser/tree/master/Data)
#### 終止上市 Suspended.pkl, 上市 codelist_2.pkl, 上櫃 codelist_4.pkl

#### 將上方檔案匯入 SQLite3 資料庫

#### 欄位 "code"(股票代號) "type"(上市twse/上櫃tpse/終止上市delisted)

> #### 確定檔案可讀

```python
# Check Pickle is correct

import pandas as pd
delisted = pd.read_pickle('Data/Suspended.pkl') #終止上市
twse = pd.read_pickle('Data/codelist_2.pkl') #上市
tpex = pd.read_pickle('Data/codelist_4.pkl') #上櫃

print(delisted.head())
print(twse.head())
print(tpex.tail())
```

> #### 建立 SQLite3 資料庫檔案

```python
# Create SQLite DB

import sqlite3

sqlite_file = 'list.sqlite'
table_stock = 'StockList'

col_pk = 'IndexColumn'
typ_pk = 'INTEGER'

col_code = 'code'
typ_code = 'TEXT'

col_type = 'type'
typ_type = 'TEXT'

conn = sqlite3.connect(sqlite_file)
c = conn.cursor()

# 建立新 Table 並含 Primary Key
c.execute('CREATE TABLE {tn} ({nf0} {ft0} PRIMARY KEY, {cn1} {ct1} NOT NULL UNIQUE, {cn2} {ct2} NOT NULL)'\
          .format(tn=table_stock, nf0=col_pk, ft0=typ_pk, cn1=col_code, ct1=typ_code, cn2=col_type, ct2=typ_type))

# IndexColumn | code     | type
# INTEGER     | TEXT     | TEXT
# PK          | NOT NULL | NOT NULL
#             | UNIQUE   |

conn.commit()
conn.close()
```

> #### 萬一執行 SQL Query 時出錯, 執行下列兩行中斷資料庫連線

```python
# Close Connection

conn.commit()
conn.close()
```

> #### 確定並將 Delisted DataFrame 轉為 str

```python
# Test Iterate delisted through DataFrame

for index, row in delisted.head().iterrows():
    print("index:", index, " row:",str(row[0][0]))
    print(type(str(row[0][0])))
```

> #### 將 Delisted 匯入資料庫

```python
# Insert Delisted Stock Codes to DB

sqlite_file = 'list.sqlite'
table_stock = 'StockList'

conn = sqlite3.connect(sqlite_file)
c = conn.cursor()

for index, row in delisted.iterrows():
    print("row:",str(row[0][0]),"delisted")
    conn.execute("INSERT OR IGNORE INTO StockList (code, type) VALUES (?,?)",(str(row[0][0]),"delisted"));
    
conn.commit()
conn.close()
```

> #### 因為上市公司是利用網站爬蟲抓取, 移除標題文字

```python
# Remove Title from Stock Codes TWSE

def RepresentsInt(s):
    try: 
        int(s)
        return True
    except ValueError:
        return False

twsli = []
    
for index, row in twse.iterrows():
    firstChar = row[0][0][0:1]
    if RepresentsInt(firstChar):
        twsli.append(row[0][0])
        
print(twsli[0:5])
```

> #### 將上市公司代號輸入資料庫

```python
sqlite_file = 'list.sqlite'
table_stock = 'StockList'

conn = sqlite3.connect(sqlite_file)
c = conn.cursor()

for item in twsli: # Which DATAFRAME
    print("row:",item,"twse")                                                      # type
    conn.execute("INSERT OR IGNORE INTO StockList (code, type) VALUES (?,?)",(item,"twse"));
    
conn.commit()
conn.close()
```

> #### 上櫃公司同上市公司

```python
# Remove Title from Stock Codes TPSE

def RepresentsInt(s):
    try: 
        int(s)
        return True
    except ValueError:
        return False

tpli = []
    
for index, row in tpex.iterrows():
    firstChar = row[0][0][0:1]
    if RepresentsInt(firstChar):
        tpli.append(row[0][0])
        
print(tpli)
```

> #### 上櫃公司代號匯入資料庫

```python
sqlite_file = 'list.sqlite'
table_stock = 'StockList'

conn = sqlite3.connect(sqlite_file)
c = conn.cursor()

for item in tpli: # Which List
    print("row:",item,"tpse")                                                      # type
    conn.execute("INSERT OR IGNORE INTO StockList (code, type) VALUES (?,?)",(item,"tpse"));
    
conn.commit()
conn.close()
```
