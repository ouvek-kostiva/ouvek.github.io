---
layout: post
title:  "自創的新技術指標 3MAD"
date:   2017-08-19 13:10:00 +0800
---

# 自創的新技術指標 3MAD

### 這個技術指標主要是利用三條均線看漲跌，不過他的界線（斜率）要自己找

### 有三個參數可以調整: 分別是哪三條均線和第一二條均線差的加權 

```
Inputs:
	Near(5),
	Middle(10),
	Far(20),
	NearMAWeight(0.5); // Slope of MA to Entry
Vars: 
	NearMA(0),
	MiddleMA(0),
	FarMA(0),
	NearDiff(0), 
	FarDiff(0),
	Diff(0);

// MA*3
NearMA = Average(Close, Near);
MiddleMA = Average(Close, Middle);
FarMA = Average(Close, Far);
//Slopes of MA
NearDiff = NearMA - MiddleMA;
FarDiff = MiddleMA - FarMA;
// Ouvek MA*3 Diff Band
Diff = (NearDiff * NearMAWeight) + FarDiff;

plot1(Diff, "OuvekMADiff");
```
