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

2. 计算生存时间

```R
# 用dod减去outtime，作为生存时间
# 参数解释：
# difftime：时间相减；
# dmy与dmy_hms：时间格式化；
# date：时间转日期
data$survival_time =difftime(dmy(data$dod),date(dmy_hms(data$outtime)),units = "days")
# survival_time 列转为数值型
data$survival_time = as.numeric(data$survival_time)
```

3. 处理生存时间

```R
# 生存时间大于等于30天状态设为存活：0，其他设为1
data$status=ifelse(data$survival_time>=30,0,1) 
```

