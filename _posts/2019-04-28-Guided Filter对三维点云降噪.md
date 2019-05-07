---
layout:     post
title:      Guided Filter对三维点云降噪
subtitle:   Guided Filter for 3D Point Cloud
date:       2019-04-28
author:     Jerry Zhao
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - Math
    - Point Cloud
---


# Guided Filter点云降噪
Guided Filter一般用来对2D图像进行降噪等处理，实际上，稍作修改后可以对3D点云进行降噪。

从Guided Filter的基本假设出发，可以推导出针对3D数据的处理方法。这里仅考虑引导数据是点云本身的情况。

首先，根据局部线性假设，有

$$
q_i=A_kp_i+b_k
$$

其中$$q_i$$是滤波后输出的三维点，$$p_i$$是当前需要滤波的点（即算法的输入），$$A_k$$是一个3x3矩阵，$$b_k$$是3x1向量。

我们希望这个局部线性模型，在$$p_i$$的领域内有最小的重建误差，即

$$
\text{arg}\min_{A_k, b_k}\sum_{j\in N(i)}(\Vert A_kp_j+b_k - p_j\Vert^2+\epsilon\Vert A_k\Vert^2)
$$


用上式分别对$$A_k$$和$$b_k$$求导，并令导数等于0，可得

$$
\begin{aligned}
&A_k(\sum_{j\in N(i)}p_jp_j^T+n\epsilon I)+b_k\sum_{j\in N(i)}p_j^T-\sum_{j\in N(i)}p_ip_i^T=0 \\
&b_k=\frac{1}{n}\sum_{j\in N(i)}p_j-A_k\frac{1}{n}\sum_{j\in N(i)}p_j=\mu_i-A_k\mu_i
\end{aligned}
$$


其中$$n$$是$$p_i$$的领域$$N(i)$$中的点数（包括$$p_i$$自己），$$\mu_i$$是$$N(i)$$中所有点的平均，将$$b_k$$带入第一个等式中，且等式两边同时除以$$n$$，有


$$
A_k(\frac{1}{n}\sum_{j\in N(i)}p_jp_j^T-\mu_i\mu_i^T+\epsilon I) = \frac{1}{n}\sum_{j\in N(i)}p_jp_j^T-\mu_i\mu_i^T
$$

注意到

$$
\frac{1}{n}\sum_{j\in N(i)}p_jp_j^T-\mu_i\mu_i^T=\Sigma_i
$$

即领域$$N(i)$$中所有点的协方差矩阵，所以最终有


$$
\begin{aligned}
A_k&=\Sigma_i(\Sigma_i+\epsilon I)^{-1} \\
b_k&=\mu_i-A_k\mu_i
\end{aligned}
$$


于是针对点云的Guided Filter算法，可概况为
1. 计算点云中某一个点$$p_i$$的领域$$N(i)$$
2. 求$$N(i)$$中所有点的均值$$\mu_i$$和协方差$$\Sigma_i$$
3. 根据上面的公式计算$$A_k$$和$$b_k$$
4. $$q_i=A_kp_i+b_k$$，输出$$q_i$$作为对点$$p_i$$的滤波结果

上面的算法实际上对Guided Filter做了一些简化，原本的Guided Filter需要得到所有包含$$p_i$$的领域对应的$$A_k$$和$$b_k$$，并对这些$$A_k,b_k$$求平均，再输出$$q_i=\bar A_kp_i+\bar b_k$$。

三维情况的Guided Filter依然保持了二维情况的优点，即对边缘或者尖锐形状的地方有较好的保护作用。平滑作用的强弱可以通过调节领域搜索时的半径$$r$$和$$\epsilon$$来改变。

其实，双边滤波（Bilateral Filter）也能用于3D点云的降噪，而Guided Filter相对于双边滤波在三维情况下的最大优势，在于不用估计点云中每个点的法线方向。

# Results
输入的带有噪声的点云：
![](https://note.youdao.com/yws/api/personal/file/WEB7a4c04da35aca32dd82685a903493321?method=download&shareKey=18135ca8d1b0ad0528f7de0d52ca2a2a)

经过两轮Guided Filter滤波后的点云：
![](https://note.youdao.com/yws/api/personal/file/WEB633d7a94d91fc350b0a12bf8dd74aff0?method=download&shareKey=2b5d931127eaaec0891d72b4896228d8)

# Notes
1. 对同一个点云进行多轮滤波（一般2轮足矣）可以大大提高降噪效果，有点类似于优化中的重复线性化思想。但过多的轮数可能造成滤波后的点云分布不均，由于计算均值的关系，每一轮滤波都会放大原来点云的非均匀性。

# Code

Python源码可见[github](https://github.com/aipiano/guided-filter-point-cloud-denoise).

# References
1. [Guided Image Filtering](http://kaiminghe.com/publications/pami12guidedfilter.pdf)
