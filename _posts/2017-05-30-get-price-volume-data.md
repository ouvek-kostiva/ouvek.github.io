---
layout: post
title:  "十年每日價量資料"
date:   2017-05-17 20:00:00 +0800
---

# 十年每日價量資料

#### [使用Yahoo/Google API取得歷史股價資料](http://lovecoding.logdown.com/posts/257928-use-yahoo-api-to-obtain-historical-stock-price-data)



> 副程式區

```python
__author__ = "Ouvek Kostiva"
__copyright__ = "2017"
__credits__ = ["Huang Hsin Yuan","Ouvek Kostiva"]
__maintainer__ = "Huang Hsin Yuan"
__email__ = "kostiva@ouvek.com"
__status__ = "Prototype"



def getListedCode(typ="twse",codeLength=4):
    import sqlite3
    sqlite_file = "list.db" #上市上櫃下市 股票代號列表 資料庫檔案
    conn = sqlite3.connect(sqlite_file)
    c = conn.cursor()
    out = conn.execute("SELECT code, type FROM StockList WHERE type LIKE ? AND length(code) == ? ",("twse",codeLength)) 
    # 取 代號4碼 類別上市 股票代號
    codeList = []
    for i, row in enumerate(out):
        codeList.append(row[0])
    conn.close()
    print("取得股票代號總數:", len(codeList)) #codeList 為取出股票代號列表
    return codeList
    
def getGoogleData(stockCode, exchange="TPE", interval="86400", duration="10Y"):
    f = "d,c,h,l,o,v" # Fields : Columns > d,c,h,l,o,v = Date + ?, Close, High, Low, Open, Volume
    import requests
    import pickle
    r = requests.get('https://www.google.com/finance/getprices?q={}&x={}&i={}&p={}&f={}'.format(stockCode,exchange,interval,duration,f))
    output = open('Pickles/{}_{}.pkl'.format(stockCode,duration), 'wb')
    print("write:","{}_{}.pkl".format(stockCode,duration))
    lines = r.text.split('\n')
    pickle.dump(lines, output) #寫入 pickle
    pklname = '{}_{}.pkl'.format(stockCode,duration)
    return pklname
    
def readPickledData(fileName):
    import os
    import pickle
    if os.path.isfile('Pickles/{}.pkl'.format(fileName)):
        pkl_file = open('Pickles/{}.pkl'.format(fileName), 'rb')
        lines = pickle.load(pkl_file)
        print("Pickled File ", fileName, " Loaded")
        return lines
    else:
        print("Filename should be like: code_duration, ex:1101_10Y")

def createDatabase(dbName,tableName):
    import os
    if os.path.isfile(dbName):
        return "Database name already exists: ", dbName
    else:    
        import sqlite3
        sqlite_file = dbName
        conn = sqlite3.connect(sqlite_file)
        c = conn.cursor()
        c.execute('CREATE TABLE {tn} (indexColumn INTEGER PRIMARY KEY, code TEXT NOT NULL, date TEXT NOT NULL, close REAL, high REAL, low REAL, open REAL, volume REAL)'.format(tn=tableName)) 
        conn.commit()
        conn.close()
        return "Database",dbName," Successfully Created!"
    
def insertData(dbName, tableName, dataList, stockCode):
    import sqlite3
    conn = sqlite3.connect(dbName)
    c = conn.cursor()
    count = 0
    for date, close, high, low, ope, vol in dataList:
        conn.execute("INSERT INTO twse (code, date, close, high, low, open, volume) VALUES (?,?,?,?,?,?,?)",(stockCode, date, close, high, low, ope, vol))
        count = count + 1
    conn.commit()
    conn.close()
    return count


def toDataList(fileName):
    import datetime as dt
    lines = readPickledData(fileName)
    del lines[0:7]
    dataList = []
    dtDate = 0;
    for ind,lin in enumerate(lines[:-1]):
        spl = lin.split(",")
        if spl[0][0] == 'a':
            print(spl[0][1:])
            actDate = dt.datetime.fromtimestamp(int(spl[0][1:])).strftime('%Y-%m-%d %H:%M:%S')
            dtDate = dt.datetime.strptime(actDate, '%Y-%m-%d %H:%M:%S')
        else:
            add = int(spl[0])
            newDate = dtDate + dt.timedelta(days=add)
            fDate = newDate.strftime('%Y-%m-%d %H:%M:%S') #fDate 日期字串
            close = spl[1]
            high = spl[2]
            low = spl[3]
            ope = spl[4]
            vol = spl[5]
            dataList.append([fDate, close, high, low, ope, vol])
    return dataList
```

> 實際執行

```python
codeList = getListedCode("twse",4) #get Stock Codes
#codeList = codeList[5:10] #get first 5 stock Codes

dbName = "PriceVolData.db"
tableName = "twse"

createDatabase("PriceVolData.db","twse") #create db named PriceVolData.db with table twse

for code in codeList:
    pklname = getGoogleData(code, exchange="TPE", interval="86400", duration="10Y")
    print(code)
    dataList = toDataList("{}_10Y".format(code)) # pkl to dataList
    count = insertData(dbName, tableName, dataList, code)

    print("Inserted : ", count, " Data")
```

> 執行結果在[這裡](https://github.com/ouvek-kostiva/stockchooser/blob/master/GoogleAPIv3.ipynb)

> 取得股票代號總數: 913
>
> write: 1101_10Y.pkl
>
> 1101
>
> Pickled File  1101_10Y  Loaded
>
> 1180503000
>
> 1266903000
>
> 1330756200
>
> 1330925400
>
> 1417411800
>
> Inserted :  2450  Data
