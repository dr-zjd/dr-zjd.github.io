---
title: 计算MIMIC数据库中患者的生存时间和生存状态
tags: 数据分析和统计
article_header:
  type: cover
  image:
---
从MIMIC数据库中直接下载后的数据并不能直接得出用作生存分析所需要的生存时间和生存状态，而是需要通过dod（死亡时间）和outtime（出ICU时间）计算。
<!--more-->

1. 加载包并读取数据

```R
# install.packages("lubridate")
library("lubridate")
data <- read.csv("mimic_time.csv")
```
