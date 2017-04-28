---
layout: post
title:  "取得所有台股股票代號（三）"
date:   2017-04-28 23:00:00 -0600
---

收集台股資料 1.3 - 找台股代號

### 來源：證交所網站下載終止上市公司列表 CSV 檔

#### 用 Pandas 將 CSV 檔匯入 DataFrame
#### Encoding 是 big5hkscs
#### 標題欄位往右挪所以用'公司名稱'可以取得代碼
#### 為方便之後使用我將 Dataframe Pickle 起來
#### 也需要研究公司終止上市原因所以先存起來

```python
import pandas as pd
import os

def getCode():
    if os.path.exists('OriginData/Suspended.csv'):
        codelist = pd.read_csv('OriginData/Suspended.csv',header=0,encoding='big5hkscs')
        c = codelist['公司名稱'] #欄位跑掉, 往右移了
        n = pd.DataFrame({'Code' : []})
        for i in c:
            print(i)
            n = n.append({'Code' : [i]},ignore_index=True)    
        n.to_pickle('Data/Suspended.pkl')
        print(n)
        return 1;
    else:
        print("Data does not exist!")
        return 0;
    
def readCode():
    print("Warning: Loading pickled data received from untrusted sources can be unsafe")
    if os.path.exists('Data/Suspended.pkl'):
        codelist = pd.read_pickle('Data/Suspended.pkl')
        print("Loading...")
        return codelist
    else:
        print("CodeList does not exist, please try again or check")
        getReturnCode = getCode()
        if 1 == getReturnCode:
            print("CodeList retrieved!")
            return readCode()
        else:
            print("Please make sure Data is in OriginData/Suspended.csv")
            return 0;

codelist = readCode()
if isinstance(codelist,pd.DataFrame):
    print(codelist['Code'].head())
else:
    print("Error! Please check you have OriginData/Suspended.csv or Data/Suspended.pkl")
```

Print 結果

```
Warning: Loading pickled data received from untrusted sources can be unsafe
CodeList does not exist, please try again or check
6702
3474
911611
911626
4733
3584
3573
3598
6286
2847
911612
913889
2361
2833
2384
911609
910069
910948
5280
911201
3061
2837
3638
8078
3697
911610
911602
9151
2315
910579
8008
3599
3080
8199
2473
911606
9104
916665
6119
5854
3367
2854
3534
2463
6012
9102
1716
2403
2341
3614
6255
1523
2391
9922
2350
2526
3063
2452
3009
2446
2381
2396
1606
2411
2494
2432
9101
3271
1311
9915
2447
2418
3020
1520
2336
2479
1107
3053
1601
6280
2469
2333
3142
2831
2827
3007
1207
9801
6004
2410
2523
3214
2807
2343
2811
2394
2487
3012
2825
1807
2335
2378
1408
2326
2407
2822
2366
2533
2422
2808
3039
2416
2470
6132
2389
1212
1204
1306
2370
1450
1462
2318
9936
2902
2435
2544
2525
1534
2490
3001
2398
2319
1224
1458
1602
2386
1228
2512
1407
1510
2828
2848
2445
1221
2521
1422
2518
2517
1222
2005
2802
2806
2819
2830
2826
1431
2304
2531
2310
2322
2301
2604
1226
2814
2843
2815
2839
2835
2016
9932
2813
2306
2818
2844
2829
2821
2824
2840
1214
2810
2846
9913
2804
2805
2803
2817
2842
1317
1230
2522
1509
2519
2202
2914
1505
2011
2019
2334
2432
         Code
0      [6702]
1      [3474]
2    [911611]
3    [911626]
4      [4733]
5      [3584]
6      [3573]
7      [3598]
8      [6286]
9      [2847]
10   [911612]
11   [913889]
12     [2361]
13     [2833]
14     [2384]
15   [911609]
16   [910069]
17   [910948]
18     [5280]
19   [911201]
20     [3061]
21     [2837]
22     [3638]
23     [8078]
24     [3697]
25   [911610]
26   [911602]
27     [9151]
28     [2315]
29   [910579]
..        ...
169    [9932]
170    [2813]
171    [2306]
172    [2818]
173    [2844]
174    [2829]
175    [2821]
176    [2824]
177    [2840]
178    [1214]
179    [2810]
180    [2846]
181    [9913]
182    [2804]
183    [2805]
184    [2803]
185    [2817]
186    [2842]
187    [1317]
188    [1230]
189    [2522]
190    [1509]
191    [2519]
192    [2202]
193    [2914]
194    [1505]
195    [2011]
196    [2019]
197    [2334]
198    [2432]

[199 rows x 1 columns]
CodeList retrieved!
Warning: Loading pickled data received from untrusted sources can be unsafe
Loading...
0      [6702]
1      [3474]
2    [911611]
3    [911626]
4      [4733]
Name: Code, dtype: object
```
