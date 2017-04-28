---
layout: post
title:  "取得所有台股股票代號（一）"
date:   2017-04-28 16:20:00 -0600
---

## 收集台股資料 1.1 - 找台股代號

### 來源一：TEJ + 今日日期

#### 匯出成 Excel 檔後轉成 CSV
#### 再將 CSV 檔用匯入成 Dataframe
#### 為方便之後使用我將 Dataframe Pickle 起來
##### TEJ 沒有 API 只能匯出 QQ

```python
import pandas as pd
import os

def getCode(): #確定CSV存在並匯入Dataframe
    if os.path.exists('OriginData/List.csv'):
        codelist = pd.read_csv('OriginData/List.csv',header=0)
        c = codelist['證券代碼']
        n = pd.DataFrame({'Code' : []})
        for i in c:
            i = i.split(" ")
            n = n.append({'Code' : [i[0]]},ignore_index=True)    
        n.to_pickle('Data/codelist.pkl')
        return 1;
    else: #
        print("Data does not exist!")
        return 0;
    
def readCode(): # 讀取Dataframe
    print("Warning: Loading pickled data received from untrusted sources can be unsafe")
    if os.path.exists('Data/codelist.pkl'):
        codelist = pd.read_pickle('Data/codelist.pkl')
        print("Loading...")
        return codelist
    else:
        print("CodeList does not exist, please try again or check")
        getReturnCode = getCode()
        if 1 == getReturnCode:
            print("CodeList retrieved!")
            return readCode()
        else:
            print("Please make sure Data is in OriginData/List.csv")
            return 0;

codelist = readCode()
if isinstance(codelist,pd.DataFrame):
    print(codelist['Code'].head()) #印出Dataframe中第一個股票代號
else:
    print("Error! Please check you have OriginData/List.csv or Data/codelist.pkl")
```

Print 結果

```
Warning: Loading pickled data received from untrusted sources can be unsafe
Loading...
0    [0050]
1    [0051]
2    [0052]
3    [0053]
4    [0054]
Name: Code, dtype: object
```
