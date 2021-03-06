---
title: 机器学习笔记（三）——斯坦福大学吴恩达（Andrew Ng）课程
date: 2017-06-15T22:33:10+08:00
draft: false
toc: true
comments: true
aliases:
  - /detail/81
  - /detail/81/
tags:
  - 机器学习
---

<script src="https://cdn.bootcss.com/mathjax/2.7.0/MathJax.js?config=default"></script>

## 九、聚类算法

***

### 1、聚类算法介绍

* 一种无监督学习
* 应用案例
	* 市场细分
	* 社交关系分析
	* 计算中心组织
	* 天文数据分析

### 2、k-均值算法

#### （1）K均值 (K-means)算法输入、输出

输入

* k，将输入分为k个簇
* x，输入的训练集，n*m矩阵，n个特征，m个数据

输出

* c，m维向量，c(i)表示第i个输入属于的簇

#### （2）K均值 (K-means)算法过程

```
随机初始化K个聚类中心 μ1, μ2, μ3, ...,μK为n维向量
重复 {
(1)簇分配：
	for i = 1 to m
		c(i) := 最接近x(i)的聚类中心uk的索引k（k的取值为1~K）
(2)移动聚类中心：
	for k = 1 to K
		uk := 所有标记为k的点的均值中心
} 直到 聚类中心不变
```

**问题：**

* 对于没有点分配给他的聚类中心，可以选择删掉他，或者随机化一个点给他
* 对于没有明确分界线的训练样本，也可以使用k均值

#### （3）K均值 (K-means)算法的代价函数（优化目标函数）

相关定义如上

$$
J(c,\mu) = \frac{1}{m} \sum\_{i=1}^{m}||x^{(i)} - \mu\_{c^{(i)}}||^2
$$
代价函数，又称之为失真函数

算法执行过程中，

* **(1)簇分配**实际上是 关于c对于J最小化的过程二\\(\mu\\)不变
* **(2)移动聚类中心** 实际上是最小化J关于\\(\mu\\)值的过程，而c不变

### 3、如何初始化K均值算法

* 选择一个K （K < m）
* 随机选择K个训练样例
* 设置μ1, μ2, μ3, ...,μK依次等于者K个样例

当k很小（2~10）
执行k均值算法10到1000次，选择出代价函数最小的那个聚类的结果作为结果

当k很大
执行1次k均值算法就能得到很好的结果

### 4、如何选择聚类数目K

#### （1）肘部算法（Elbow Method）

绘制出代价函数和聚类数目K的函数：

若得到一条类似于人类肘部的函数（单调减）
那么这个肘点就是我们选择的K；

若得到的是比较平滑的线那么，就很难选择合适的K。

#### （2）根据你的业务目的（下游目的、后续目的）选择K

比如为选择T恤的尺寸聚类，根据具体的市场需求选择K

### 5、维数约简（数据压缩）

#### （1）、含义

* 舍弃一些冗余的特征变量，将2维转化为1维，或者3位转换为2维以此类推

#### （2）作用

* 调高算法运行速度
* 可视化数据

#### （3）、位数简约算法

* 主成分分析（PCA）算法

#### （4）PCA的公式表述

寻找一个低维度的面，使得所有输入X到这个面的距离（投影误差）最短

要将n维样本约简到k维，寻找k个k维的向量u1，u2，..., uk；然后将样本数据映射到这k个向量展开的子空间上，最小化投影误差

看起来类似于线性回归，但是不全是

#### （5）PCA的实现过程

**数据预处理**

* 拿到某组m个无标签样本的训练集，首先进行均值归一化（mean normalization），或者说是特征缩放，参见[特征缩放](#https://www.rectcircle.cn/detail/64#（1）特征缩放法)

假设，要将n维样本约简到k维。

**首先**计算协方差矩阵\\(\Sigma\\)，为n\*n的矩阵
$$
\Sigma = \frac{1}{m} \sum\_{i=1}^{m}(x^{(i)})(x^{(i)})^{T}
$$
向量化后
$$
\Sigma = \frac{1}{m} X^TX
$$

**然后**找到一个能计算\\(\Sigma\\)奇异值分解(eigenvectors)的库函数
在octave中

```
[U,S,V] = svd(Sigma);
%或者eig(Sigma)
Ureduce = U(:,1:k);
%U 是n*n的矩阵，每一列组成的向量就是我们需要u1, u2, ...,uk。（只抽取前k个）
%叫做Uredure，n*k的矩阵
```

**最后计算**

$$
z^{(i)} = U\_{redure}^T x^{(i)}
$$

在octave中，用于计算一个映射

```
z = Ureduce'*x
```

* \\(x^{(i)}\\)是n\*1维的向量
* \\(z^{(i)}\\)是k\*1维的向量
* 最后得到的z就是将维后的样本

或者在octave中直接计算，用于计算训练集

```
Z =  X * Ureduce;
```

* Z 是m\*k的矩阵
* X 是m\*n的矩阵

#### （5）从压缩过的数据恢复到原数据

$$
X\_{approx}^{(i)} = U\_{reduce} z^{(i)}
$$

#### （6）如何选择PCA算法中K（主成分的数量）

**概念**
平均投影误差：
$$
\frac{1}{m} \sum\_{i=1}^{m} ||x^{(i)} - x\_{approx}^{(i)} ||^2
$$

数据总变化
$$
\frac{1}{m} \sum\_{i=1}^{m} ||x^{(i)}||^2
$$

得到一个计算差异性的公式
$$
\frac{ \frac{1}{m} \sum\_{i=1}^{m} ||x^{(i)} - x\_{approx}^{(i)}||^2 }{ \frac{1}{m} \sum\_{i=1}^{m} ||x^{(i)}||^2 } \le 0.01
$$
这样程PCA算法保留了原数据的99%的差异性

**步骤**
选择一个要保留的差异性
若使用的是 octave

```
[U,S,V] = svd(Sigma);
```

其中的S是一个仅主对角线有数据，其他位置为零的n\*n的矩阵
$$
得到一个计算差异性的公式 = 1 - \frac{ \sum\_{i=1}^{k} S\_{ii} } { \sum\_{i=1}^{n} S\_{ii} }
$$

#### （7）使用PCA的应用和建议

* 调高训练效率——对于有监督学习训练集(x,y)，忽略标签y，得到无标签数据集x，使用PCA算法降维得到z（同时得到Ureduce），在和标签y组合起来的到新的有标签训练集(z,y)来进行训练。当有新的样本到来，在使用Ureduce进行降维后，在应用到模型上。
* 压缩数据
* 数据可视化
* 最好不要用来防止过拟合
* 一开始最好不要使用PCA优化算法，直到算法运行的十分之慢在尝试使用

## 十、异常检测

***

### 1、异常检测介绍

例子：
一个飞机引擎制造商，当你生产的飞机引擎，从生产线上流出时，你需要进行QA (质量控制测试)。而作为这个测试的一部分，你测量了飞机引擎的一些特征变量。比如，你可能测量了，引擎运转时产生的热量，或者引擎的振动等等。采集这些特征变量，你就有了一个数据集 从x(1)到x(m) 如果你生产了m个引擎的话，这都是你的无标签数据。

异常检测问题可以定义如下：
我们假设，后来有一天，你有一个新的飞机引擎，从生产线上流出。而你的新飞机引擎，有特征变量x-test。所谓的异常检测问题就是，我们希望知道，这个新的飞机引擎是否有某种异常。

因此，当我们建立了x的概率模型之后，对于新的飞机引擎，也就是x-test 如果概率p低于阈值ε，那么就将其标记为异常。

```
p(Xtest) < ε 标记为异常
p(Xtest) >= ε 正常
```

**用途**

* 检查网站用户是否被盗号，或者是异常用户
* 工业生产领域，预估产品是否存在问题
* 计算机集群，异常节点检测

### 2、高斯分布（正态分布）

**公式**
$$
p(x; \mu, \sigma^2) = \frac{1}{\sqrt{2\pi}\sigma} exp (- \frac{(x-\mu)^2}{2\sigma ^2})
$$
**说明**

* 图形为一个钟形曲线
* \\(\mu\\)，表示图形中心的位置，表示数据集的均值
* \\(\sigma\\)，表示图形的高度和宽度，表示数据集的标准差

### 3、使用高斯分布实现异常检测算法

**说明**

训练集：m个无标签训练样本的训练集，每个样本为n维向量特征，所以训练集X为m\*n的矩阵

那么假设对于每个特征都符合正态分布，且相互独立，则有：
$$
p(x) = p(x\_1, \mu\_1, \sigma\_1^2 )p(x\_2, \mu\_2, \sigma\_2^2 )...p(x\_n, \mu\_n, \sigma\_n^2) =  \prod\_{j=1}^n p(x\_j,\mu\_j, \sigma\_j^2)
$$

* x为n维向量

**步骤**

* 选择合适的特征组成训练集
* 计算出\\(\mu\_1, ..., \mu\_n, \sigma\_1, ..., \sigma\_n\\)
* 给一个新的样本x，计算\\(p(x)\\)
	* 如果\\(p(x)<	\epsilon\\) ，异常
	* 否则正常

### 4、如何构建一个异常检测系统

#### （1）评价异常检测算法的性能

假设我们有一些带有标签的数据，y=0代表正常，y=1代表异常

将这些样本分为训练集，交叉验证集（CV），测试集，比例为：6:2:2

使用训练集训练异常检测算法。然后在交叉验证集检测。检测方法见[6、偏斜类（skewed classes）](78#6、偏斜类（skewed classes）)
同时选择\\(\epsilon \\)

#### （2）如何选择异常检测和监督学习算法

既然训练集带有标签，为什么不选择监督学习的算法呢？如何选择？

以下情况选择**异常检测算法**

* 正样本的数目很少（0~20个，监督学习不足以学到很多特征）
* 负样本的数目很多

以下情况选择**监督学习算法**

* 正负样本都很多

#### （3）如何选择特征变量

绘制每个特种变量的直方图，观察是否是高斯分布（octave语法：hist(x,50)）
对于是高斯分布的变量，直接送入算法
对于不是高斯分布的变量，通过一个变换（例如\\(log(x+c),x^c\\)），拟合高斯分布

然后通过误差分析检测，分析特征变量选择，添加新的特征，这个特征变量可以是由现存的特征经过合理组合得到的

### 5、多元高斯分布

参数：\\(\\mu\\)是n维向量， \\(\Sigma\\) 为n\*n的矩阵，表示协方差（见[5、维数约简（数据压缩）](#5、维数约简（数据压缩）)）

公式：

$$
p(x; \mu, \Sigma) = \frac{1}{(2\pi)^{\frac{n}{2}} | \Sigma|^{\frac{1}{2}} } exp(-\frac{1}{2}(x-\mu)^{T} \Sigma^{-1}(x-\mu))
$$

* 其中\\(|\Sigma|\\) 表示行列式，在octave中使用det函数计算

#### （1）算法实现

给予训练集：\\({ x^{(1)},x^{(2)},...,x^{(m)} }\\)，每个x为n维向量

**计算\\(\mu, \Sigma\\)：**
$$
\mu = \frac{1}{m} \sum\_{i=1}^m x^{(i)}
$$
$$
\Sigma = \frac{1}{m} \sum\_{i=1}^{m}(x^{(i)} - \mu)(x^{(i)} - \mu)^{T}
$$

**对于一个新的样本x计算：**
$$
p(x; \mu, \Sigma) = \frac{1}{(2\pi)^{\frac{n}{2}} | \Sigma|^{\frac{1}{2}} } exp(-\frac{1}{2}(x-\mu)^{T} \Sigma^{-1}(x-\mu))
$$

**标记异常：**
$$
p(x) < \epsilon
$$

#### （2）普通和多元高斯分布的关系

普通高斯分布是多元高斯分布的特例

#### （2）如何选择两种模型

**普通模型**

* 更加常用，但是如果要捕捉两个特征变量有相关性，需要构造新的特征变量例如(x = cpu负载/内存)
* 运算量小
* m可以很小

**多元高斯模型**

* 不常用，当时可以捕捉到特征变量之间的相关性
* 运算量大
* 必须满足m>n或者说，\\(\Sigma\\)可逆，一般来说满足 m>10n，避免出现线性相关性的特征

## 十一、推荐系统

***

研究推荐系统的动机：

* 科技公司内最大的应用
* 可以作为学习算法自动选取随机变量的例子

### 1、预测电影评分

给你一些电影和用户，用户对某些电影进行打分，要求通过这些数据，预测出用户没打过分电影的分数

|电影名|用户A|用户B|用户C|用户D|浪漫指数x1|动作指数x2|
|-----|-----|-----|-----|-----|-----|-----|
|电影1|5|5|0|0|0.9|0|
|电影2|5|?|?|0|1.0|0.01|
|电影3|?|4|0|?|0.99|0|
|电影4|0|0|5|4|0.1|1.0|
|电影5|0|0|5|?|0|0.9|

用户可以给电影评分0~5

**定义**

* \\(n\_u\\)表示用户的数目
* \\(n\_m\\)表示电影的数目
* \\(r(i,j)=1\\)表示用户j评价的电影i
* \\(y^{(i,j)}\\) 表示用户j给电影i的评分
* \\(m^{(j)}\\)表示用户j评价过的电影的数目

### 2、基于内容的推荐系统

假设每部电影都含有几个特征来描述，例如电影有x1表示电影的浪漫指数，x2表示电影的动作性，所以 \\(x^{(i)}\\)表示第i个电影的特征，此时为3维向量（加上x0=1）

对于每个用户j，训练参数\\(\theta^{(j)}\\) 为3（根据内容的特征数得到）维向量，训练的过称为：将每个已评价的电影的特征为训练集的X，电影的评分为y，X-y构成的有便签训练集，使用监督学习（如线性回归）算法训练处参数。再带入未评分的电影的特征得到预测的评分。

假设使用线性回归算法，数学上的描述为：
最小化问题：
给一组\\(x^{(i)},...,x^{(n\_m)}\\)，来学习 \\(\theta^{(1)}, ...,\theta^{(nu)}\\)
$$
min \frac{1}{2m^{(j)}} \sum\_{i:r(i,j)=1} ( (\theta^{(j)})^T (x^{(i)}) - y^{(i,j)} )^2 + \frac{\lambda}{2m^{(j)}} \sum\_{k=1}^n (\theta\_k^{(j)})^2
$$
梯度下降法
$$
\displaystyle \min\_{\theta^{(1)},\dots,\theta^{(n\\\_m)}} \frac{1}{2}\sum\_{j=1}^{n\\\_u}\sum\_{i:r(i,j)=1}\left((\theta^{(j)})^T x^{(i)}-y^{(i,j)}\right)^2 + \frac{\lambda}{2}\sum\_{j=1}^{n\\\_u}\sum\_{k=1}^n(x\_k^{(j)})^2
$$

其他参考线性回归

### 3、协同过滤法构建推荐算法

特点：自动学习需要的特征

|电影名|用户A\\(\theta^{(1)}\\)|用户B\\(\theta^{(2)}\\)|用户C\\(\theta^{(3)}\\)|用户D\\(\theta^{(4)}\\)|浪漫指数x1|动作指数x2|
|-----|-----|-----|-----|-----|-----|-----|
|\\(x^{(i)}\\)电影1|5|5|0|0|?|?|
|\\(x^{(i)}\\)电影2|5|?|?|0|?|?|
|\\(x^{(i)}\\)电影3|?|4|0|?|?|?|
|\\(x^{(i)}\\)电影4|0|0|5|4|?|?|
|\\(x^{(i)}\\)电影5|0|0|5|?|?|?|

假设每部电影的特征未知，例如电影有x1表示电影的浪漫指数，x2表示电影的动作性，但是\\(x^{(i)}\\)是未知的。
假设我们采访得知每个用户对电影的各种特征的评分，\\(\theta^{(j)}\\)，例如：
\\(\theta^{(1)} = [0;5;0]\\)，表示，用户A对浪漫性的电影给5分，对动作性电影给0分，同理\\(\theta^{(2)} = [0;5;0]\\)，\\(\theta^{(3)} = [0;0;5]\\)，\\(\theta^{(4)} = [0;0;5]\\) 。
那么对于电影1，\\((\theta^{(1)})^T X^{(1)}=5\\)，\\((\theta^{(2)})^T X^{(1)}=5\\)，\\((\theta^{(3)})^T X^{(1)}=0\\)，\\((\theta^{(4)})^T X^{(1)}=0\\)，可以看出电影1的x1 = 1.0，x2=0

最小化问题：
给一组\\(\theta^{(1)}, ...,\theta^{(n\_u)}\\) 来学习\\(x^{(i)},...,x^{(n\_m)}\\)
$$
min \frac{1}{2} \sum\_{j:r(i,j)=1} ( (\theta^{(j)})^T - y^{(i,j)} )^2 + \frac{\lambda}{2m^{(j)}} \sum\_{k=1}^n (\theta\_k^{(j)})^2
$$

梯度下降的算法：
$$
\displaystyle \min\_{x^{(1)},\dots,x^{(n\\\_m)}} \frac{1}{2}\sum\_{i=1}^{n\\\_m}\sum\_{j:r(i,j)=1}\left((\theta^{(j)})^T x^{(i)}-y^{(i,j)}\right)^2 + \frac{\lambda}{2}\sum\_{i=1}^{n\\\_m}\sum\_{k=1}^n(x\_k^{(i)})^2
$$

而，与根据内容的推荐相比，求解的方程相同。

**协同过滤法**
随机初始化\\(\theta\\) -> 得到 x，再得到 \\(\theta\\)，依次迭代

也就是说：
**a.**给一组\\(x^{(i)},...,x^{(n\_m)}\\)，来学习 \\(\theta^{(1)}, ...,\theta^{(nu)}\\)
$$
\displaystyle \min\_{\theta^{(1)},\dots,\theta^{(n\\\_m)}} \frac{1}{2}\sum\_{j=1}^{n\\\_u}\sum\_{i:r(i,j)=1}\left((\theta^{(j)})^T x^{(i)}-y^{(i,j)}\right)^2 + \frac{\lambda}{2}\sum\_{j=1}^{n\\\_u}\sum\_{k=1}^n(\theta\_k^{(j)})^2
$$

**b.**给一组\\(\theta^{(1)}, ...,\theta^{(n\_u)}\\) 来学习\\(x^{(i)},...,x^{(n\_m)}\\)
$$
\displaystyle \min\_{x^{(1)},\dots,x^{(n\\\_m)}} \frac{1}{2}\sum\_{i=1}^{n\\\_m}\sum\_{j:r(i,j)=1}\left((\theta^{(j)})^T x^{(i)}-y^{(i,j)}\right)^2 + \frac{\lambda}{2}\sum\_{i=1}^{n\\\_m}\sum\_{k=1}^n(x\_k^{(i)})^2
$$

**a.** sum-每个用户(sum-关于该用户的每个电影的评分)
**b.** sum-每个电影(sum-关于该电影的每个用户的评分)

合并起来，为一个最小化问题
$$
J(x^{(i)},...,x^{(n\_m)},\theta^{(1)}, ...,\theta^{(nu)}) =  \frac{1}{2} \sum\_{j:r(i,j)=1} \left((\theta^{(j)})^T x^{(i)}-y^{(i,j)}\right)^2 + \frac{\lambda}{2}\sum\_{i=1}^{n\\\_m}\sum\_{k=1}^n(x\_k^{(i)})^2  + \frac{\lambda}{2}\sum\_{j=1}^{n\\\_u}\sum\_{k=1}^n(\theta\_k^{(j)})^2
$$

**注意**
不需要添加：x0=1

**算法步骤**

* 随机初始化化\\(x^{(i)},...,x^{(n\_m)},\theta^{(1)}, ...,\theta^{(nu)}\\)
* 使用梯度下降或者高级算法迭代求参数
假设使用梯度下降法

\\(i=1,...,n\_m\\)

$$
x\_k^{(i)} = x\_k^{(i)} - \alpha( \sum\_{j:r(i,j)=1} ( (\theta^{(j)})^T x^{(i)} - y^{(i,j)} )\theta\_k^{(j)} + \lambda x\_k^{(i)})
$$
\\(j=1,...,n\_u\\)
$$
\theta\_k^{(j)} = \theta\_k^{(j)} - \alpha( \sum\_{i:r(i,j)=1} ( (\theta^{(j)})^T x^{(i)} - y^{(i,j)} )x\_k^{(i)} + \lambda \theta\_k^{(i)})
$$

* 对于一个用户的参数 \\(\theta \\) 和得到的电影的特征的\\(x\\) 得到用户未评分的电影的得分：\\(\theta^Tx\\)

### 4、向量化协同过滤算法实现

#### （1）参数向量化

|电影名|用户A|用户B|用户C|用户D|浪漫指数x1|动作指数x2|
|-----|-----|-----|-----|-----|-----|-----|
|电影1|5|5|0|0|0.9|0|
|电影2|5|?|?|0|1.0|0.01|
|电影3|?|4|0|?|0.99|0|
|电影4|0|0|5|4|0.1|1.0|
|电影5|0|0|5|?|0|0.9|

* n_m = 5，表示产品的数目
* n_u = 4，表示用户的数目
* \\(y^{(i,j)}\\) 用户j对用产品j的评分

$$
Y = \begin{bmatrix}
5&5&0&0\\\\
5&?&?&0\\\\
?&4&0&?\\\\
0&0&5&4\\\\
0&0&5&?
\end{bmatrix}
$$

Y矩阵的另一种描述：
$$
Y = \begin{bmatrix}
     (\theta^{(1)})^T (x^{(1)}) & (\theta^{(2)})^T (x^{(1)})  & \cdots & (\theta^{(n\_u})^T (x^{(1)}) \\\\
     (\theta^{(1)})^T (x^{(2)}) & (\theta^{(2)})^T (x^{(2)})  & \cdots & (\theta^{(n\_u})^T (x^{(2)}) \\\\
     \vdots  & \vdots & \ddots & \vdots \\\\
     (\theta^{(1)})^T (x^{(n\_m)}) & (\theta^{(2)})^T (x^{(n\_m)})  & \cdots & (\theta^{(n\_u})^T (x^{(n\_m)}) \\\\
\end{bmatrix}
$$

* \\(\theta^{(j)}\\)表示用户j的模型
* \\(x^{(i)}\\)表示产品i的特征
* \\( (\theta^{(j)})^T (x^{(i)}) \\)表示用户j对用产品j的评分

所有产品的特征的矩阵
$$
X = \begin{bmatrix}
     \cdots & (x^{(1)})T & \cdots \\\\
		 \cdots & (x^{(2)})T & \cdots \\\\
		 \vdots & \vdots & \vdots\\\\
		 \cdots & (x^{(n\_m)})T & \cdots \\\\
\end{bmatrix}
$$

* 每一行表示一个产品的特征

所有用户的模型的矩阵

$$
\Theta = \begin{bmatrix}
     \cdots & (\theta^{(1)})T & \cdots \\\\
		 \cdots & (\theta^{(2)})T & \cdots \\\\
		 \vdots & \vdots & \vdots\\\\
		 \cdots & (\theta^{(n\_u}))T & \cdots \\\\
\end{bmatrix}
$$

这样Y矩阵可以这么描述

$$
Y = X\Theta^{T}
$$

#### （2） 低秩矩阵分解（协同过滤算法）

对于每一个产品i，我们学习一个特征\\(x^{(i)}\\)（n维向量）

如何找到一些产品j与产品i相关（相似、可能会购买）？
$$
small || x^{(i)} - x^{(i)} ||
$$

#### （3）均值归一化

不使用归一化，那么对于某些用户没有评价过的产品将得到零分

过程
原矩阵
$$
Y = \begin{bmatrix}
5&5&0&0&?\\\\
5&?&?&0&?\\\\
?&4&0&?&?\\\\
0&0&5&4&?\\\\
0&0&5&?&?
\end{bmatrix}
$$
均值向量
$$
\mu = \begin{bmatrix}
2\.5\\\\
2\.5\\\\
2\\\\
2\.25\\\\
1\.25
\end{bmatrix}
$$
归一化后的Y矩阵
$$
Y = \begin{bmatrix}
2\.5&2.5&-2.5&-2.5&?\\\\
2\.5&?&?&-2.5&?\\\\
?&2&-2&?&?\\\\
-2.25&-2.25&2.75&1.75&?\\\\
-1.25&-1.25&3.75&-1.25&?
\end{bmatrix}
$$

对于也测结果，用户j对产品i的评分
$$
(\theta^{(j)})^T(x^{(i)}) + \mu\_i
$$

## 十二、大规模机器学习

***

### 1、大数据集机器学习

* 在处理大数据训练集之前，先随机抽取少量训练样本，绘制学习曲线，观察出现的是高偏差还是高方差
	* 高偏差（欠拟合），使用大数据对模型训练有益
	* 高方差（过拟合），使用大数据对训练模型没有显著提升，所以要提高特征变量数

### 2、随机梯度下降

以线性回归为例：

算法描述：
1、打乱数据集
2、
重复 {
&nbsp;&nbsp;  for i:= 1,...,m{
&nbsp;&nbsp;&nbsp;&nbsp;    \\(\theta\_j := \theta\_j - \alpha( h\_\theta^{(i)} -y^{(i)} )x\_j^{(i)} \\) (for j = 0,...,n)
&nbsp;&nbsp;  }
} 直到收敛

* 一般重复1~10就足够了，这取决于样本大小m的值

### 3、小批量梯度下降

算法描述：
假设b = 10，m = 1000
重复 {
&nbsp;&nbsp; for i = 1,11,21,31,...,991 {
&nbsp;&nbsp;&nbsp;&nbsp;  \\(\theta\_j := \theta\_j - \alpha\frac{1}{m} \sum\_{k=i}^{i+9} (h\_\theta(x^{(k)})-y^{(k)})x\_j^{(k)} \\) (for every j=0,..,n)
&nbsp;&nbsp; }
} 直到收敛

* b=2~100

### 4、如何检测大数据下算法是否收敛

在随机梯度下降中

定义：
$$
cost(\theta,(x^{(i)},y^{(i)})) = \frac{1}{2}(h\_\theta(x^{(i)}) - y^{(i)})^2
$$

在学习过程中，计算\\(cost(\theta,(x^{(i)},y^{(i)}))\\)，在使用\\(x^{(i)},y^{(i)})\\)更新\\(\theta\\)之前

最后为了检查收敛性，每进行1000（假设）次迭代，就计算出前一步计算的cost函数的均值，绘制出图形

动态选定学习参数\\(\alpha\\)，例如
$$
\alpha = \frac{const1}{iterationNumber + const2}
$$

### 5、在线学习

#### （1）算法描述

（假设使用逻辑回归）
一直重复 {
&nbsp;&nbsp; 当用户给予一组新的数据(x,y)
&nbsp;&nbsp; 使用(x,y)更新\\(\theta\\):
&nbsp;&nbsp;&nbsp;&nbsp;\\(\theta\_j := \theta\_j - \alpha(h\_\theta^{(x)}-y)x\_j\\) (j=0,...,n)
}

* 实际上每各个训练数据只被使用了一次，让后丢弃，对于拥有大流量的网站，会拥有很好的效果

#### （2）在线学习的例子

产品搜索（预估点击率CTR）

* 用户输入关键字，提供最佳的10个推荐结果
* x = 产品的特征，有许多搜索关键词构成。
* y = 1 表示用户将会认同这个产品，否则 y = 0

### 6、映射约减和数据并行性

假设我们要处理线性回归问题，m = 400，使用梯度下降

* 将数据等切分为几份（4份，每份100个）
* 将数据子集分给4台设备计算求和项\\( temp\_j = \sum h\_\theta^{(i)} -y^{(i)} )x\_j^{(i)} \\)
* 将4个结果临时传送给一台机器，进行合并操作，完成\\(\theta \\) 的更新

运用映射约简的前提：

* 将机器学习的问题描述成训练样本的某种求和（包含\\(\sum\\)项）
