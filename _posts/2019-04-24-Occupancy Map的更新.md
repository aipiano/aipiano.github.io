---
layout:     post
title:      Occupancy Map的更新
subtitle:   Update Occupancy Map
date:       2019-04-24
author:     Jerry Zhao
header-img: img/post-bg-gridmap.jpg
catalog: true
tags:
    - SLAM
    - Robotics
---


# Occupancy Map
将空间划分为一个个小网格（cell），每个cell中存储cell内是否有障碍物的概率，这样的地图称为Occupancy Map，由于所有cell构成一个网，所以也称为Grid Map。

以2D Occupancy Map为例，每个cell中都有一个概率，那整个地图可以看做一个描述障碍物分布的概率分布（严格要归一化后才是概率分布）。构建地图的过程，就是根据传感器（比如Lider）的观测，更新这个概率分布。

设每个cell的概率相对于其他cell是独立的，更新地图的概率分布就简化为更新每个cell的概率。即在已知1到t时刻所有观测的情况下，该cell中有障碍物的概率，记为

$$
p(m_i|z_{1:t})
$$


# Odds
考虑到更新的方便，每个cell中实际存储的是概率对应的Odds。

Odds定义为有障碍物的概率与无障碍物概率的比，即

$$
o(m_i|z_{1:t})=\frac{p(m_i|z_{1:t})}{1 - p(m_i|z_{1:t})} = \frac{p(m_i|z_{1:t})}{p(\neg m_i|z_{1:t})}
$$

已知Odds，也很容易反算出对应的概率。可以将Odds理解为一个映射，将值域在$$[0, 1]$$之间的概率映射到$$[0,+\infty)$$。

# Odds的更新

根据贝叶斯公式，有

$$
p(m_i|z_{1:t})=\frac{p(z_t|m_i, z_{1:t-1})p(m_i|z_{1:t-1})}{p(z_t|z_{1:t-1})}
$$

根据Markov假设，已知障碍物$$m_i$$的情况下，观测结果与之前的观测无关，则

$$
p(m_i|z_{1:t})=\frac{p(z_t|m_i)p(m_i|z_{1:t-1})}{p(z_t|z_{1:t-1})}
$$


于是根据Odds的定义，可得

$$
\begin{aligned}
o(m_i|z_{1:t})&=\frac{p(m_i|z_{1:t})}{p(\neg m_i|z_{1:t})} \\
&=\frac{p(z_t|m_i)p(m_i|z_{1:t-1})}{p(z_t|\neg m_i)p(\neg m_i|z_{1:t-1})} \\
&=\frac{p(z_t|m_i)}{p(z_t|\neg m_i)}\cdot o(m_i|z_{1:t-1}) \\
&=\frac{p(m_i|z_t)p(z_t)/p(m_i)}{p(\neg m_i|z_t)p(z_t)/p(\neg m_i)} \cdot o(m_i|z_{1:t-1}) \ \ \ \ \text{再次利用了贝叶斯公式} \\
&=o(m_i|z_t)\cdot o(m_i|z_{1:t-1})\cdot \underbrace{\frac{p(\neg m_i)}{p(m_i)}}_{\text{prior}}
\end{aligned}
$$


在没有观测的情况下，一般假设cell有障碍物的概率为0.5，即$$p(m_i)=p(\neg m_i)=0.5$$，所以最终有


$$
o(m_i|z_{1:t})=o(m_i|z_t)\cdot o(m_i|z_{1:t-1})
$$


即更新某个cell的Odds时，只需用新的观测结果对应的Odds，乘以该cell原本的Odds即可。

比如Lider击中cell中的物体，并返回一个距离值，这时可认为cell中有障碍物的概率为0.9，即$$p(m_i|hit)=0.9$$，对应的Odds就是$$o(m_i|hit)=9$$，如果激光没有击中任何东西，则其传输路径上的所有cell存在障碍物的概率就较低，设为$$p(m_i|loss)=0.2$$，对应的Odds即$$o(m_i|loss)=0.25$$。这样，Odds的更新就变成了根据激光的返回情况，选择乘以9还是0.25。

# 对数Odds
更进一步的，可以对Odds取对数，每个cell中保存的也是Odds的对数值，则Odds的更新变为

$$
\log o(m_i|z_{1:t})=\log o(m_i|z_t) + \log o(m_i|z_{1:t-1})
$$

即将乘法变为了加法，在$$\log o(m_i|z_t)$$是几个已知常数值的情况下（如之前Lider的例子），这种方式更高效。

