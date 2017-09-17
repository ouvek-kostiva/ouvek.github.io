---
layout: post
title:  "技術分析程式碼"
date:   2017-09-17 17:30:00 +0800
---

# MACD EMA SMA RSI

```python
##########################################################################################
# Technical Analysis Data
# MACD, RSI, SMA, EMA
##########################################################################################

def getTicker():
    import sqlite3
    dbName = "list.db"
    tableName = "StockList"
    conn = sqlite3.connect(dbName)
    c = conn.cursor()
    sqlquery = "SELECT code, type FROM " + str(tableName)
    out = conn.execute(sqlquery)
    stockList = []
    for code, typ in out:
        stockList.append([code, typ])
    conn.close()
    return stockList

def getData(ticker):
    import sqlite3
    dbName = "PriceVolData.db"
    tableName = "twse"
    conn = sqlite3.connect(dbName)
    c = conn.cursor()
    sqlquery = "SELECT code, close, date FROM twse WHERE code=" + str(ticker)
    out = conn.execute(sqlquery)
    dalist = []
    for code, close, date in out:
        dalist.append([code, close, date[:10]])
    conn.close()
    return dalist

def moving_average(x, n, type='simple'):
    from numpy import array
    import numpy as np
    #compute an n period moving average.
    #type is 'simple' | 'exponential'
    x = np.asarray(x)
    if type == 'simple':
        weights = np.ones(n)
    else:
        weights = np.exp(np.linspace(-1., 0., n))
    weights /= weights.sum()
    a = np.convolve(x, weights, mode='full')[:len(x)]
    a[:n] = a[n]
    return a

def moving_average_convergence(x, nslow=26, nfast=12):
    from numpy import array
    import numpy as np
    #compute the MACD (Moving Average Convergence/Divergence) using a fast and slow exponential moving avg'
    #return value is emaslow, emafast, macd which are len(x) arrays
    emaslow = moving_average(x, nslow, type='exponential')
    emafast = moving_average(x, nfast, type='exponential')
    return emaslow, emafast, emafast - emaslow

def relative_strength(prices, n=14):
    from numpy import array
    import numpy as np
    #compute the n period relative strength indicator
    #http://stockcharts.com/school/doku.php?id=chart_school:glossary_r#relativestrengthindex
    #http://www.investopedia.com/terms/r/rsi.asp
    deltas = np.diff(prices)
    seed = deltas[:n+1]
    up = seed[seed >= 0].sum()/n
    down = -seed[seed < 0].sum()/n
    rs = up/down
    rsi = np.zeros_like(prices)
    rsi[:n] = 100. - 100./(1. + rs)
    for i in range(n, len(prices)):
        delta = deltas[i - 1]  # cause the diff is 1 shorter
        if delta > 0:
            upval = delta
            downval = 0.
        else:
            upval = 0.
            downval = -delta
        up = (up*(n - 1) + upval)/n
        down = (down*(n - 1) + downval)/n
        rs = up/down
        rsi[i] = 100. - 100./(1. + rs)
    return rsi

def getTA(dalist, nslow=20, nfast=5, nema=10):
    from numpy import array
    import numpy as np
    fh = array(dalist) # Data(Ticker,Price,Date) to Numpy Array
    prices = fh[:,1].astype(np.float) # Price from Str to float
    emaslow, emafast, macd = moving_average_convergence(prices, nslow=nslow, nfast=nfast)
    rsi = relative_strength(prices) # RSI
    revMACD = list(reversed(macd))
    revRSI = list(reversed(rsi))
    revDalist = list(reversed(dalist))
    dalistTA = []
    for i in range(len(dalist)):
        dalistTA.append([dalist[i][0],dalist[i][1],dalist[i][2],revMACD[i],revRSI[i]])
    reversed(dalistTA)
    return dalistTA
```

### 測試結果

```python
stockList = getTicker() #從 list.db 中取得股票代號
dalist = getData(1301) #從 PriceVolData.db 中取得股票日期及收盤價
dalistTA = getTA(dalist, nslow=20, nfast=5, nema=10) # 計算 MACD 和 RSI
print(dalistTA[0]) # 顯示

['1301', 67.7, '2007-05-31', -0.89156536415006826, 50.782443498991547]
```
