---
layout: post
title:  "透過 Python 定期下載 CSV 資料"
date:   2017-11-08 10:30:00 +0800
---

# 以下載行政院原子能委員會全國環境輻射偵測資料為例 (每 10 分鐘)

```python
def download():
    import csv
    import requests
    CSV_URL = 'http://www.aec.gov.tw/open/gammamonitor.csv'
    with requests.Session() as s:
        download = s.get(CSV_URL)
        download = download.content
        cr = csv.reader(download.splitlines(), delimiter=',')
        my_list = list(cr)
        with open("gammadata.csv", "a") as fp:
            wr = csv.writer(fp)
            for line in my_list:
                wr.writerow(line[1:])   
import time
from datetime import datetime
while True:
    download()
    print("Dowloaded at:",str(datetime.now()))
    time.sleep(10*60)
```
