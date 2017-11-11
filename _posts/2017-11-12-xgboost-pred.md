---
layout: post
title:  "XGBoost 訓練完成後測試"
date:   2017-11-12 00:30:00 +0800
---

## 這是利用已經訓練完成的 XGBoost 模型進行測試

### 1 使用到的副程式

```python
def getTWSEdata(stockCode="1301", monthDate="20170801"):
    # monthDate = 20170801
    import requests
    import pickle
    import os
    import json
    r = requests.get('http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date={}&stockNo={}'.format(monthDate,stockCode))
    print('http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date={}&stockNo={}'.format(monthDate,stockCode))
    data = json.loads(r.text)
    prices = data['data']
    return prices
    #"代號  名稱 日期       成交股數   成交金額     開盤價 最高價 最低價 收盤價 漲跌差 成交筆數"

def getThisMonth(daysAgo=31):
    from datetime import datetime, date, time, timedelta
    today = datetime.today()
    first = today.replace(day=1)
    lastMonth = first - timedelta(days=daysAgo)
    dateStr = lastMonth.strftime("%Y%m01")
    return dateStr

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

def getTA(fh, nslow=20, nfast=5, nema=10):
    from numpy import array
    import numpy as np
    fh = array(fh)
    prices = fh.astype(np.float) # Price from Str to float
    emaslow, emafast, macd = moving_average_convergence(prices, nslow=nslow, nfast=nfast)
    rsi = relative_strength(prices) # RSI
    revMACD = list(reversed(macd))
    revRSI = list(reversed(rsi))
    revDalist = list(reversed(fh))
    dalistTA = []
    for i in range(len(fh)):
        dalistTA.append([fh[i],revMACD[i],revRSI[i]])
    reversed(dalistTA)
    return dalistTA

def predictAnswer(code,file,thresh=0.5):
    import xgboost as xgb
    nameofMODEL = 'data5day'+str(code)+'.model'
    bst = xgb.Booster({'nthread': 4})
    bst.load_model(nameofMODEL)
    dtest = xgb.DMatrix(file)
    preds = bst.predict(dtest)
    return preds
```

### 2 執行

```python
code=2399

dateStr = getThisMonth(331)
prices = getTWSEdata(stockCode=code, monthDate=dateStr)
dateStr = getThisMonth(301)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))
dateStr = getThisMonth(271)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))
dateStr = getThisMonth(241)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))
dateStr = getThisMonth(211)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))
dateStr = getThisMonth(181)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))
dateStr = getThisMonth(151)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))
dateStr = getThisMonth(121)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))
dateStr = getThisMonth(91)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))
dateStr = getThisMonth(61)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))
dateStr = getThisMonth(31)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))
dateStr = getThisMonth(0)
prices.extend(getTWSEdata(stockCode=code, monthDate=dateStr))

CloseList = []
predictions = []

for i,val in enumerate(prices):
    CloseList.append(prices[i][6])
dalistTA = getTA(CloseList, nslow=20, nfast=5, nema=10)

for i,val in enumerate(dalistTA):
    dtest = "1:"+str(dalistTA[i][0])+" 2:"+str(dalistTA[i][1])+" 3:"+str(dalistTA[i][2])+'\n'
    with open('temp.csv', 'w') as f:
        f.write(dtest)
    preds = predictAnswer(code,'temp.csv')
    print(i," ",preds," ",dalistTA[i][0])
    predictions.append(preds)
    
print("Average:",sum(predictions)/float(len(predictions)),"")
```

### 3 執行結果

```python
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20161201&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20170101&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20170201&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20170301&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20170401&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20170501&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20170601&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20170701&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20170801&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20170901&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20171001&stockNo=2399
http://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20171101&stockNo=2399
0   [ 0.67158175]   8.10
1   [ 0.67158175]   8.25
2   [ 0.69630325]   8.22
3   [ 0.70128816]   8.29
4   [ 0.70128816]   8.26
5   [ 0.70128816]   8.20
6   [ 0.70128816]   8.20
7   [ 0.70128816]   8.48
8   [ 0.70128816]   8.47
9   [ 0.70128816]   8.47
10   [ 0.70128816]   8.30
11   [ 0.70128816]   8.40
12   [ 0.70128816]   8.31
13   [ 0.70128816]   8.35
14   [ 0.70128816]   8.35
15   [ 0.70128816]   8.35
16   [ 0.70128816]   8.25
17   [ 0.70128816]   8.18
18   [ 0.70128816]   8.25
19   [ 0.70128816]   8.28
20   [ 0.49493903]   8.25
21   [ 0.69630325]   8.33
22   [ 0.67158175]   8.29
23   [ 0.68418682]   8.30
24   [ 0.62202936]   8.43
25   [ 0.62202936]   8.33
26   [ 0.6022957]   8.37
27   [ 0.68703157]   8.62
28   [ 0.68703157]   8.57
29   [ 0.68703157]   8.54
30   [ 0.68703157]   8.52
31   [ 0.68703157]   8.44
32   [ 0.68703157]   8.34
33   [ 0.68703157]   8.36
34   [ 0.68703157]   8.48
35   [ 0.68703157]   8.89
36   [ 0.68703157]   8.85
37   [ 0.68703157]   8.73
38   [ 0.68703157]   8.70
39   [ 0.68703157]   8.88
40   [ 0.68703157]   8.90
41   [ 0.68703157]   8.92
42   [ 0.6022957]   9.05
43   [ 0.68418682]   8.95
44   [ 0.49493903]   8.93
45   [ 0.70128816]   8.77
46   [ 0.70128816]   8.75
47   [ 0.70128816]   8.85
48   [ 0.70128816]   8.94
49   [ 0.70128816]   8.99
50   [ 0.70128816]   8.95
51   [ 0.70128816]   8.97
52   [ 0.70128816]   9.04
53   [ 0.70128816]   9.16
54   [ 0.70128816]   9.26
55   [ 0.70128816]   9.26
56   [ 0.70128816]   9.20
57   [ 0.49493903]   9.19
58   [ 0.67158175]   9.12
59   [ 0.69630325]   9.15
60   [ 0.62202936]   9.17
61   [ 0.62202936]   9.10
62   [ 0.62202936]   9.12
63   [ 0.62202936]   9.10
64   [ 0.68418682]   9.01
65   [ 0.69630325]   8.95
66   [ 0.69630325]   8.98
67   [ 0.69630325]   8.99
68   [ 0.69630325]   9.01
69   [ 0.67158175]   8.99
70   [ 0.67694241]   9.01
71   [ 0.70128816]   9.24
72   [ 0.70128816]   9.30
73   [ 0.70128816]   9.36
74   [ 0.70128816]   9.74
75   [ 0.70128816]   9.61
76   [ 0.70128816]   9.59
77   [ 0.70128816]   9.50
78   [ 0.70128816]   9.55
79   [ 0.70128816]   9.60
80   [ 0.70128816]   9.50
81   [ 0.70128816]   9.53
82   [ 0.70128816]   9.70
83   [ 0.70128816]   9.50
84   [ 0.70128816]   9.60
85   [ 0.70128816]   9.52
86   [ 0.70128816]   9.50
87   [ 0.70128816]   10.00
88   [ 0.70128816]   11.00
89   [ 0.70128816]   11.30
90   [ 0.70128816]   10.50
91   [ 0.70128816]   10.50
92   [ 0.70128816]   11.00
93   [ 0.70128816]   11.05
94   [ 0.70128816]   12.15
95   [ 0.70128816]   11.95
96   [ 0.70128816]   12.05
97   [ 0.70128816]   11.55
98   [ 0.70128816]   11.45
99   [ 0.70128816]   11.50
100   [ 0.70128816]   11.20
101   [ 0.70128816]   10.45
102   [ 0.70128816]   10.50
103   [ 0.70128816]   11.10
104   [ 0.70128816]   10.70
105   [ 0.70128816]   10.60
106   [ 0.70128816]   10.40
107   [ 0.70128816]   10.45
108   [ 0.70128816]   10.60
109   [ 0.70128816]   10.70
110   [ 0.70128816]   11.70
111   [ 0.70128816]   11.50
112   [ 0.70128816]   11.85
113   [ 0.70128816]   11.85
114   [ 0.70128816]   11.70
115   [ 0.70128816]   11.35
116   [ 0.70128816]   11.35
117   [ 0.70128816]   11.30
118   [ 0.70128816]   11.30
119   [ 0.70128816]   11.40
120   [ 0.69630325]   12.50
121   [ 0.67158175]   12.20
122   [ 0.68418682]   12.80
123   [ 0.68418682]   12.70
124   [ 0.68418682]   13.00
125   [ 0.67158175]   13.25
126   [ 0.67158175]   13.40
127   [ 0.67158175]   12.95
128   [ 0.70128816]   13.00
129   [ 0.70128816]   13.00
130   [ 0.70128816]   13.60
131   [ 0.70128816]   13.50
132   [ 0.70128816]   13.65
133   [ 0.70128816]   13.45
134   [ 0.70128816]   13.80
135   [ 0.70128816]   13.95
136   [ 0.70128816]   14.00
137   [ 0.70128816]   14.60
138   [ 0.70128816]   14.60
139   [ 0.70128816]   14.75
140   [ 0.70128816]   15.20
141   [ 0.70128816]   14.85
142   [ 0.70128816]   15.00
143   [ 0.70128816]   14.95
144   [ 0.70128816]   16.40
145   [ 0.70128816]   16.30
146   [ 0.70128816]   16.00
147   [ 0.70128816]   17.45
148   [ 0.70128816]   17.60
149   [ 0.70128816]   16.60
150   [ 0.70128816]   17.15
151   [ 0.70128816]   17.60
152   [ 0.70128816]   18.45
153   [ 0.70128816]   18.35
154   [ 0.70128816]   17.00
155   [ 0.70128816]   17.20
156   [ 0.70128816]   17.00
157   [ 0.70128816]   16.75
158   [ 0.70128816]   16.90
159   [ 0.70128816]   16.70
160   [ 0.67694241]   16.60
161   [ 0.67694241]   16.60
162   [ 0.49493903]   16.80
163   [ 0.49493903]   17.35
164   [ 0.49493903]   17.20
165   [ 0.49493903]   16.85
166   [ 0.70128816]   16.80
167   [ 0.70128816]   15.95
168   [ 0.70128816]   15.60
169   [ 0.70128816]   16.85
170   [ 0.70128816]   16.10
171   [ 0.70128816]   16.35
172   [ 0.70128816]   16.65
173   [ 0.70128816]   16.80
174   [ 0.70128816]   17.05
175   [ 0.70128816]   16.75
176   [ 0.70128816]   16.80
177   [ 0.70128816]   16.60
178   [ 0.70128816]   16.60
179   [ 0.70128816]   16.95
180   [ 0.70128816]   16.95
181   [ 0.70128816]   16.75
182   [ 0.70128816]   16.80
183   [ 0.70128816]   16.95
184   [ 0.70128816]   17.30
185   [ 0.70128816]   17.05
186   [ 0.70128816]   16.20
187   [ 0.70128816]   15.40
188   [ 0.70128816]   15.40
189   [ 0.70128816]   15.30
190   [ 0.70128816]   14.85
191   [ 0.70128816]   14.90
192   [ 0.70128816]   15.00
193   [ 0.70128816]   15.00
194   [ 0.70128816]   13.65
195   [ 0.70128816]   14.10
196   [ 0.70128816]   14.25
197   [ 0.70128816]   14.05
198   [ 0.70128816]   14.25
199   [ 0.70128816]   14.00
200   [ 0.70128816]   13.75
201   [ 0.70128816]   13.30
202   [ 0.70128816]   13.30
203   [ 0.70128816]   13.30
204   [ 0.70128816]   13.30
205   [ 0.70128816]   13.40
206   [ 0.70128816]   13.75
207   [ 0.70128816]   13.60
208   [ 0.70128816]   13.75
209   [ 0.70128816]   13.75
210   [ 0.70128816]   13.85
211   [ 0.67694241]   13.80
212   [ 0.67694241]   14.05
213   [ 0.70128816]   14.00
214   [ 0.70128816]   14.30
215   [ 0.70128816]   14.05
216   [ 0.70128816]   14.05
217   [ 0.70128816]   13.95
218   [ 0.70128816]   13.75
219   [ 0.70128816]   14.05
220   [ 0.70128816]   14.00
221   [ 0.70128816]   14.00
222   [ 0.70128816]   13.80
223   [ 0.70128816]   14.00
224   [ 0.70128816]   14.55
225   [ 0.70128816]   14.30
226   [ 0.67694241]   14.00
227   [ 0.67694241]   13.85
228   [ 0.67694241]   13.70
229   [ 0.67694241]   13.70
230   [ 0.67694241]   13.40
231   [ 0.67694241]   13.40
232   [ 0.67694241]   13.35
Average: [ 0.68826753]
```
