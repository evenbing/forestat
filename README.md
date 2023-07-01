# <div align="center"><strong>使用 Forestat 评估森林质量</strong></div>

<p align="right"><strong>Forestat version:</strong> 1.0.2</p>
<p align="right"><strong>Date:</strong> 07/01/2023 </p>
<br>

*`forestat`* 是基于中国林业科学研究院资源信息研究所（Institute of Forest Resource Information Techniques, Chinese Academy of Forestry）的`天然林立地质量评价方法`[<sup>[1]</sup>](#citation)开发的R包。实现的功能包括天然林基于林分高生长的立地等级划分，树高模型、断面积生长模型、生物量生长模型的建立，林分现实生产力与潜在生产力的计算。使用 *`forestat`* 包可以为精准提升森林质量提供可靠依据。

<div align="center">

[English](README.en-US.md) | [简体中文](README.md)
<br>

</div>

## <div align="center">1 概述</div>

*`forestat`* 包实现了天然林基于林分高生长的立地等级划分，树高模型、断面积生长模型、生物量生长模型的建立，林分现实生产力与潜在生产力的计算。其中，树高模型可用Richard模型、Logistic模型、korf模型、Gompertz模型、Weibull模型和Schumacher模型构建，断面积生长模型和生物量生长模型仅可用Richard模型构建。*`forestat`* 包依赖于天然林的样地数据，包中带有一份样例数据。

### 1.1 *forestat* 流程图

<div align="center">
  <img width="70%" src="forestat/vignettes/img/flowchart.png">
  <p>图 1. <i>forestat</i>工作流程图</p>
</div>

### 1.2 *forestat* 依赖的R包

| **Package** | **Download Link**                          |
| ----------- | ------------------------------------------ |
| dplyr       | https://CRAN.R-project.org/package=dplyr   |
| ggplot2     | https://CRAN.R-project.org/package=ggplot2 |
| nlme        | https://CRAN.R-project.org/package=nlme    |


## <div align="center">2 安装</div>

### 2.1 从CRAN或GitHub安装
在 R 中使用以下命令从 [CRAN](https://CRAN.R-project.org/package=forestat) 安装 *`forestat`*  ：

```R
# 安装forestat
install.packages("forestat")
```

当然，你也可以在 R 中使用以下命令从 [GitHub](https://github.com/caf-ifrit/forestat) 安装 *`forestat`*  ：

```R
# 安装devtools
install.packages("devtools")

# 安装forestat
devtools::install_github("caf-ifrit/forestat/forestat")
```

### 2.2 加载  *forestat*

```R
library(forestat)
```

## <div align="center">3 快速开始</div>

本节展示的是快速完成天然林立地质量评估的完整步骤，使用的数据是包中自带的`forestData`样例数据。

```R
# 加载包中 forestData 样例数据
data("forestData")

# 基于 forestData 数据建立模型，返回一个 forestData 类对象
forestData <- class.plot(forestData, model = "Richards",
                         interval = 5, number = 5,
                         H_start=c(a=20,b=0.05,c=1.0))

# 绘制断面积生长模型散点图
plot(forestData,model.type = "BA",plot.type = "Scatter",
     title = "桦木阔叶混断面积生长模型散点图")

# 计算 forestData 对象的潜在生产力
forestData <- potential.productivity(forestData)

# 计算 forestData 对象的现实生产力
forestData <- realized.productivity(forestData)

# 获取 forestData 对象的汇总数据
summary(forestData)
```

## <div align="center">4 详细教程</div>

<details>
<summary style="font-size:21px;"><strong>4.1 建立模型</strong></summary>

<br>
<details>
<summary style="font-size:18px;"><strong>4.1.1 自定义数据</strong></summary>

为了建立一个准确的模型，好的数据是不可或缺的，在 *`forestat`* 包中内置了一个经过清洗的样例数据，可以通过如下命令加载查看样例数据：

```R
# 加载包中 forestData 样例数据
data("forestData")

# 或者读取包中 forestat.csv 样例数据
forestData <- read.csv(system.file("extdata", "forestData.csv", package = "forestat"))

# 筛选 forestData 样例数据中ID、code、AGE、H、S、BA 和 Bio字段
# 并查看前6行数据
head(dplyr::select(forestData,ID,code,AGE,H,S,BA,Bio))

# 输出
  ID code AGE   H         S       BA       Bio
1  1    1  13 2.0 152.67461 4.899382 32.671551
2  2    1  15 3.5  68.23825 1.387268  5.698105
3  3    1  20 4.2 128.32683 3.388492 22.631467
4  4    1  19 4.2 204.93928 4.375324 18.913886
5  5    1  13 4.2  95.69713 1.904063  6.511951
6  6    1  25 4.7 153.69393 4.129810 28.024739
```

当然，你也可以选择加载自定义数据：

```R
# 加载自定义数据
forestData <- read.csv("/path/to/your/folder/your_file.csv")
```

自定义数据中`ID（样地ID）`、`code（样地林分类型代码）`、`AGE（林分平均年龄）`、`H（林分平均高）`是必须字段，用以建立`树高模型（H-model）`，并绘制相关示例图。

`S（林分密度指数）`、`BA（林分断面积）`、`Bio（林分生物量）`是可选的字段，用以建立`断面积生长模型（BA-model）`与`生物量生长模型（Bio-model）`。

在后续的潜在生产力和现实生产力计算中，断面积生长模型与生物量生长模型是必须的。也就是自定义数据如果缺少`S`、`BA`和`Bio`字段将无法计算潜在生产力和现实生产力。

<div align="center">
  <img width="70%" src="forestat/vignettes/img/forestData.png">
  <p>图 2. 自定义数据格式要求</p>
</div>

</details>

<br>
<details>
<summary style="font-size:18px;"><strong>4.1.2 构建林分生长模型</strong></summary>
<div id="4.1.2"></div>

数据加载后，*`forestat`* 将使用`class.plot()`函数构建林分生长模型，如果自定义数据中同时包含`ID、code、AGE、H、S、BA、Bio`字段，则会同时构建`树高模型、断面积生长模型、生物量生长模型`，如果只包含`ID、code、AGE、H`字段，则只会构建`树高模型`。

```R
# 选用 Richards 模型构建林分生长模型
# interval=5表示初始树高分类的林分年龄区间设置为5，number=5表示初始树高分类数最多为5，maxiter=1000表示拟合模型的最大次数为1000
# 树高模型拟合的初始参数H_start默认为c(a=20,b=0.05,c=1.0)
# 断面积生长模型拟合的初始参数BA_start默认为c(a=80, b=0.0001, c=8, d=0.1)
# 生物量生长模型拟合的初始参数Bio_start默认为c(a=450, b=0.0001, c=12, d=0.1)
forestData <- class.plot(forestData, model = "Richards",
                         interval = 5, number = 5, maxiter=1000,
                         H_start=c(a=20,b=0.05,c=1.0),
                         BA_start = c(a=80, b=0.0001, c=8, d=0.1),
                         Bio_start=c(a=450, b=0.0001, c=12, d=0.1))
```

其中，`model`为构建树高模型时选用的模型，可在`"Logistic"、"Richards"、"Korf"、"Gompertz"、"Weibull"、"Schumacher"`模型中任选一个，断面积生长模型和生物量生长模型默认使用Richard模型构建。`interval`为初始树高分类的林分年龄区间，number为初始树高分类数的最大值，`maxiter`为最大拟合次数。`H_start` 是拟合树高模型的初始参数，`BA_start` 是拟合断面积生长模型的初始参数，`H_start` 是拟合生物量生长模型的初始参数，当拟合出现错误时，可以多尝试一些初始参数作为尝试。

由`class.plot()`函数返回的结果为`forestData` 对象，包括`Input`（输入数据和树高分级结果）、`Hmodel`（H-model: 树高模型）、`BAmodel`（BA-model: 断面积生长模型）、`Biomodel`（Bio-model: 生物量生长模型）以及`output`（模型参数）。

<div align="center">
  <img width="70%" src="forestat/vignettes/img/forestDataObj.png">
  <p>图 3. forestData对象结构</p>
</div>

</details>

<br>
<details>
<summary style="font-size:18px;"><strong>4.1.3 获取汇总数据</strong></summary>
<div id="4.1.3"></div>

为了解模型的建立情况，可以使用`summary(forestData)`函数获取`forestData`对象汇总数据。该函数返回`summary.forestData`对象并将相关数据输出至屏幕。

输出的第一段为输入数据的汇总，第二、三、四段分别为`H-model`、`BA-model`、`Bio-model`的参数及其精简报告。

```R
summary(forestData)
```

```R
# 输出
# 第一段
       H               S                 BA              Bio         
 Min.   : 2.00   Min.   :  68.24   Min.   : 1.387   Min.   :  5.698  
 1st Qu.: 8.10   1st Qu.: 366.37   1st Qu.: 9.641   1st Qu.: 52.326  
 Median :10.30   Median : 494.76   Median :13.667   Median : 78.502  
 Mean   :10.62   Mean   : 522.53   Mean   :14.516   Mean   : 90.229  
 3rd Qu.:12.90   3rd Qu.: 661.84   3rd Qu.:18.750   3rd Qu.:115.636  
 Max.   :19.10   Max.   :1540.13   Max.   :45.749   Max.   :344.412  

# 第二段
H-model Parameters:
Nonlinear mixed-effects model fit by maximum likelihood
  Model: H ~ 1.3 + a * (1 - exp(-b * AGE))^c 
  Data: data 
       AIC      BIC    logLik
  728.4366 747.2782 -359.2183

Random effects:
 Formula: a ~ 1 | LASTGROUP
               a  Residual
StdDev: 3.767163 0.7035752

Fixed effects:  a + b + c ~ 1 
      Value Std.Error  DF  t-value p-value
a 12.185054 1.7050081 313 7.146625       0
b  0.037840 0.0043682 313 8.662536       0
c  0.761367 0.0769441 313 9.895060       0
 Correlation: 
  a      b     
b -0.110       
c -0.093  0.946

Standardized Within-Group Residuals:
         Min           Q1          Med           Q3          Max 
-3.858592084 -0.719253472  0.007120413  0.761123585  3.375793806 

Number of Observations: 320
Number of Groups: 5 

Concise Parameter Report:
Model Coefficients:
       a1       a2       a3       a4       a5          b         c
 7.013778 9.575677 11.90324 14.67456 17.75801 0.03783956 0.7613666

Model Evaluations:
           pe      RMSE        R2       Var       TRE      AIC      BIC    logLik
 -0.006484677 0.6980625 0.9543312 0.4887767 0.3960163 728.4366 747.2782 -359.2183

Model Formulas:
                                       Func                  Spe
 model1:H ~ 1.3 + a * (1 - exp(-b * AGE))^c model1:pdDiag(a ~ 1)

# 第三段（与第二段数据格式相似）
BA-model Parameters:

# 此处省略
......

# 第四段（与第二段数据格式相似）
Bio-model Parameters:

# 此处省略
......
```

</details>
</details>

<br>
<details>
<summary style="font-size:21px;"><strong>4.2 绘制图像</strong></summary>

经过[4.1.2](#4.1.2) `class.plot()`函数构建林分生长模型后，就可以使用`plot()`函数绘制图像。

其中，`model.type`为绘图使用的模型，可以选择`H`（树高模型）、`BA`（断面积生长模型）或者`Bio`（生物量生长模型）。`plot.type`为绘图的类型，可以选择`Curve`（曲线图）、`Residual`（残差图）、`Scatter_Curve`（散点曲线图）、`Scatter`（散点图）。`xlab`、`ylab`、`legend.lab`、`title`分别为`x轴标题`、`y轴标题`、`图例`、`图像标题`。

```R
# 绘制树高模型的曲线图
plot(forestData,model.type="H",
     plot.type="Curve",
     xlab="年龄(year)",ylab="树高(m)",legend.lab="立地等级",
     title="桦木阔叶混树高模型曲线图")

# 绘制断面积生长模型散点图
plot(forestData,model.type="BA",
     plot.type="Scatter",
     xlab="年龄(year)",ylab="树高(m)",legend.lab="立地等级",
     title="桦木阔叶混断面积生长模型散点图")
```

不同的`plot.type`绘制的样图如图4所示：

<div align="center">
  <img width="100%" src="forestat/vignettes/img/plot-1.png">
  <img width="100%" src="forestat/vignettes/img/plot-2.png">
  <p>图 4. 不同的plot.type绘制的样图</p>
</div>

</details>

<br>
<details>
<summary style="font-size:21px;"><strong>4.3 计算林分潜在生产力</strong></summary>

经过[4.1.2](#4.1.2) `class.plot()`函数构建林分生长模型后，就可以使用`potential.productivity()`函数计算林分潜在生产力。在计算之前，要求`forestData` 对象中`BA-model`和`Bio-model`已经建立。

```R
forestData <- potential.productivity(forestData, code=1,
                                     age.min=5,age.max=150,
                                     left=0.05, right=100,
                                     e=1e-05, maxiter = 50) 
```

其中，参数`code`为计算潜在生产力使用的林分类型代码。`age.min`和`age.max`分别为林分年龄的最小值和最大值，潜在生产力的计算会在最小值和最大值的区间中进行。`left`和`right`为拟合模型的初始参数，当拟合出现错误时，可以多尝试一些初始参数作为尝试。`e`为拟合模型的精度，当残差低于`e`时，认为模型收敛并停止拟合。`maxiter`为拟合模型的最大次数，当拟合次数等于`maxiter`时，认为模型收敛并停止拟合。

<br>
<details>
<summary style="font-size:18px;"><strong>4.3.1 潜在生产力输出数据说明</strong></summary>

计算结束后，可以使用如下命令查看并输出结果：

```R
library(dplyr)
forestData$potential.productivity %>% head(.)
```

```R
# 输出
    Max_GI   Max_MI       N1       D1       S0       S1       G0       G1       M0       M1 LASTGROUP AGE
1 3.761042 19.28856 9092.212 6.920702 1507.847 1655.607 30.44160 34.20265 108.8879 128.1764         1   5
2 3.188422 16.87999 8155.087 7.271688 1485.048 1607.682 30.67953 33.86795 114.2059 131.0859         1   6
3 2.767240 15.05141 7430.771 7.589146 1464.360 1568.882 30.84594 33.61318 118.7456 133.7970         1   7
4 2.444777 13.61090 6848.971 7.880151 1445.173 1536.067 30.95813 33.40291 122.6777 136.2886         1   8
5 2.189472 12.44383 6371.915 8.149517 1428.018 1508.285 31.04766 33.23713 126.2039 138.6477         1   9
6 1.982947 11.47713 5968.835 8.400940 1411.699 1483.484 31.10237 33.08531 129.3253 140.8024         1  10
```

输出结果中，各字段含义如下：

`Max_GI`：最大年生长量

`Max_MI`：林分生物量最大年生长量

`N1`：达到潜在生长量对应的林分株数

`D1`：达到潜在生长量对应的林分平均直径

`S0`： 初始林分密度指数

`S1`：达到潜在生长量对应的最佳林分密度指数

G0：初始林分每公顷断面积

`G1`：达到潜在生长量对应的林分每公顷断面积(1年以后)

`M0`：初始林分每公顷生物量

`M1`：达到潜在生长量对应的林分每公顷生物量

</details>
</details>

<br>
<details>
<summary style="font-size:20px;"><strong>4.4 计算林分现实生产力</strong></summary>

经过[4.1.2](#4.1.2) `class.plot()`函数构建林分生长模型后，可以使用`realized.productivity()`函数计算林分现实生产力。在计算之前，要求`forestData` 对象中`BA model`和`Bio model`已经建立。

```R
forestData <- realized.productivity(forestData, 
                                   left=0.05, right=100)
```

其中，参数`left`与`right`是拟合模型的初始参数，当拟合出现错误时，可以多尝试一些初始参数作为尝试。

<br>
<details>
<summary style="font-size:18px;"><strong>4.4.1 现实生产力输出数据说明</strong></summary>

计算结束后，可以使用如下命令查看并输出结果：

```R
library(dplyr)
forestData$realized.productivity %>% head(.)
```

```R
# 输出
  code ID AGE   H class0 LASTGROUP       BA         S       Bio        BAI        VI
1    1  1  13 2.0      1         1 4.899382 152.67461 32.671551 0.19401957 1.0350981
2    1  2  15 3.5      1         1 1.387268  68.23825  5.698105 0.07413948 0.3923068
3    1  3  20 4.2      1         1 3.388492 128.32683 22.631467 0.11162375 0.6491135
4    1  4  19 4.2      1         1 4.375324 204.93928 18.913886 0.19004576 1.1177796
5    1  5  13 4.2      2         1 1.904063  95.69713  6.511951 0.11925821 0.6218927
6    1  6  25 4.7      1         1 4.129810 153.69393 28.024739 0.11106428 0.6846125
```

输出结果中，各字段含义如下：

`BAI`：生物量现实生产力

`VI`：生物量潜在生产力

</details>
</details>

<br>
<details>
<summary style="font-size:20px;"><strong>4.5 潜在生产力和现实生产力数据详情</strong></summary>

在得到林分潜在生产力与现实生产力后，可以使用`summary(forestData)`函数获取`forestData`对象汇总数据。该函数返回`summary.forestData`对象并将相关数据输出至屏幕。

输出的前四段在[4.1.3](#4.1.3)中已经介绍，第五段为潜在生产力与现实生产力数据详情。

```R
summary(forestData)
```

```R
# 输出
# 第一段
       H               S                 BA              Bio         
 Min.   : 2.00   Min.   :  68.24   Min.   : 1.387   Min.   :  5.698  
 
# 此处省略
......

# 第五段
     Max_GI           Max_MI      
 Min.   :0.1396   Min.   : 1.201  
 1st Qu.:0.1966   1st Qu.: 1.779  
 Median :0.2894   Median : 2.492  
 Mean   :0.5224   Mean   : 3.871  
 3rd Qu.:0.5423   3rd Qu.: 4.277  
 Max.   :4.2408   Max.   :25.399  

      BAI                VI        
 Min.   :0.06736   Min.   :0.3923  
 1st Qu.:0.16967   1st Qu.:1.3487  
 Median :0.23468   Median :1.8711  
 Mean   :0.26202   Mean   :2.0260  
 3rd Qu.:0.31483   3rd Qu.:2.4330  
 Max.   :1.02810   Max.   :6.8336 
```

</details>

## <div align="center">5 引用</div>

<div id="citation"></div>

```txt
@article{lei2018methodology,
  title={Methodology and applications of site quality assessment based on potential mean annual increment.},
  author={Lei Xiangdong, Fu Liyong, Li Haikui, Li Yutang, Tang Shouzheng},
  journal={Scientia Silvae Sinicae},
  volume={54},
  number={12},
  pages={116-126},
  year={2018},
  publisher={The Chinese Society of Forestry}
}
```
