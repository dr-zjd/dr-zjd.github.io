---
title: R语言统计分析—中介效应分析
tags: 数据分析和统计
article_header:
  type: cover
  image:
---

### 一、中介效应的简单思考

临床研究大多聚焦于研究暴露因素与结局变量之间的相关性，对于已经报道过的暴露与结局之间，可以尝试找一个中介变量从解释暴露因素是如何导致结局变量的角度出发，利用因果中介效应分析进一步探究暴露与结局变量之间的因果关系（类似于基础研究中的机制研究）。

### 二、R语言实现

1. 加载包与样例数据

```R
# install.packages("plyr")
# install.packages("mediation")
library("plyr")
library("mediation")
# 读取数据，其中gender，enthnicity，status为分类变量，其他为连续变量
data <- read.csv("mediate_data.csv")
```
![20220318001.png](https://s1.imagehub.cc/images/2022/03/18/20220318001.png)

### 2. 线性回归中介效应分析

```R
# 设置随机种子，使结果可以复现
set.seed(12345)
# 自变量X: "age", 中介变量M: "gender", 因变量Y: "status"
mx = lm(gender ~ age, data = data)
yx = lm(status ~ age + gender, data = data)
# mediate函数中的参数介绍：treat: 自变量X，mediator: 中介变量M
# boot=T, sims=1000: 采用bootstrap检验,采样数为1000
mediate = mediate(mx, yx, treat = 'age', mediator = 'gender',
                   boot = T, sims = 1000)
summary(mediate);plot(mediate); #作图
```

![20220318002.png](https://s1.imagehub.cc/images/2022/03/18/20220318002.png)

关于这个结果的解读可参考 https://neurosci.feishu.cn/wiki/wikcn0GxLwOTBtIfJ3esAcNC8Cg

### 3. 加入协变量的线性回归中介效应分析

```R
# 设置随机种子
set.seed(12345)
# 按照lm(m ~ x + 协变量1 + 协变量2)格式加入协变量即可
mx = lm(gender ~ age + height + sofa, data = data)
# 按照lm(y ~ x + m + 协变量1 + 协变量2)格式加入协变量即可
yx = lm(status ~ age + gender + height + sofa, data = data)
mediate = mediate(mx, yx, treat = 'age', mediator = 'gender', sims = 1000, boot = T)
summary(mediate)
```

![20220318003.png](https://s1.imagehub.cc/images/2022/03/18/20220318003.png)

### 4. 批量中介效应分析

当我们确定要研究的Y与X后，如何快速的分析出收集的变量中是否存在介导X与Y的关系的中介变量?

以下代码分析出来的结果是未纳入协变量的结果，建议用于探索性分析，如果发现有符合临床意义的阳性结果，可根据第三步中的代码纳入协变量再进一步分析。

```R
# 定义函数， 复制粘贴即可
setRefClass("TS",fields = list(m="character",x="character",xm="formula",xy="formula"))
test <- new("TS")
batch_mediate <- function(ms,y,xs,df) {
  test$m <-ms
  test$x <-xs
  test$xm<-as.formula(paste0(ms,"~",xs))
  model_xm <- lm(test$xm, data = df) 
  test$xy<-as.formula(paste0(y,"~",ms,"+",xs)) 
  model_xy <- lm(test$xy, data = df)
  model_med <- mediate(model_xm, model_xy, treat = test$x, mediator = test$m, sims = 1000, boot = T)
  med_sum <- summary(model_med)
  ACME <- med_sum$d0
  ACME_P <- med_sum$d0.p
  ADE <- med_sum$z0
  ADE_P <- med_sum$z0.p
  TE <- (ACME+ADE)
  Prop_Mediated <- ACME/TE
  result <- data.frame('Characteristics'=ms, 'ACME' = ACME, 'ACME_P' = ACME_P, 'ADE' = ADE, 'ADE_P' = ADE_P, 'Total_Effect' = TE, 'Prop_Mediated' = Prop_Mediated)
  return (result)
}
```

```R
# 设置需要探索是否为中介变量M的列名
cols <-c("gender", "ethnicity", "height", "sapsii", "sofa", "heartrate")
# 设置随机种子
set.seed(12345)
# 修改y=因变量名，x=自变量名, df=分析数据
res <- lapply(cols,batch_mediate,y="status",x="age",df=data)
# 生成结果
res <- ldply(res,data.frame);
```




