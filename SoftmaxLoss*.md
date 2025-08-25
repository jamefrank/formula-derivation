# *Softmax Loss

## Softmax Loss

![img](https://i-blog.csdnimg.cn/blog_migrate/621ea3c8c2e8a782f1a80f5823475010.png#pic_center)
$$
\begin{aligned}
SL &= -{1 \over N}\sum_{i=1}^N\sum_{k=1}^Ky_{i,k}*log({e^{z_{i,k}} \over \sum_{j=1}^Ke^{z_{i,j}}}) \\
&= -{1 \over N}\sum_{i=1}^N\sum_{k=1}^Ky_{i,k}*log({e^{||W_{y_k}||||x_i||cos(\theta_{y_{i,k}})} \over \sum_{j=1}^K e^{||W_{y_{i,j}}||||x_i||cos(\theta_{y_{i,j}})}}) \\
\end{aligned}
$$

- $y_{i,k}$表示第i个样本属于第k个类别的真是标签,当样本i属于类别k时,$y_{i,k}=1$,否则$y_{i,k}=0$
- $z_{i,k}$是样本i关于类别k的logits,**全连接的输出结果就叫做logits**
- N表示样本数
- K表示类别数
- 全连接层可以认为是基于距离函数为余弦相似度的线性分类器

## L-Softmax

$$
LSL= -{1 \over N}\sum_{i=1}^N\sum_{k=1}^Ky_{i,k}*log({e^{||W_{y_{i,k}}||||x_i||\psi(\theta_{y_{i,k}})} \over \sum_{j\neq y_{i,j}}e^{||W_{y_{i,j}}||||x_i||cos(\theta_{y_{i,j}})}+e^{||W_{y_{i,k}}||||x_i||\psi(\theta_{y_{i,k}})}})
$$


$$
\psi(\theta) = \begin{cases} 
cos(m\theta) & 0 \leq \theta \leq {\pi \over m} \\
D(\theta) & {\pi \over m} < \theta \leq \pi
\end{cases}
$$
论文中设计如下
$$
\psi{\theta} = (-1)^k cos(m\theta) - 2k \\
\theta \in [{k\pi \over m}, {(k+1)\pi \over m}] \\
k \in [0, m-1]
$$
其中 m 为正整数,m越大,分类边距越大

## A-Softmax

不同于L-Softmax,将分类器权重W归一化为1
$$
ASL= -{1 \over N}\sum_{i=1}^N\sum_{k=1}^Ky_{i,k}*log({e^{||x_i||\psi(\theta_{y_{i,k}})} \over \sum_{j\neq y_{i,j}}e^{||x_i||cos(\theta_{y_{i,j}})}+e^{||x_i||\psi(\theta_{y_{i,k}})}}) \\
\psi{\theta} = (-1)^k cos(m\theta) - 2k \\
\theta \in [{k\pi \over m}, {(k+1)\pi \over m}] \\
k \in [0, m-1]
$$

## AM-Softmax

Additive Angular Margin Loss

更改为了新的  $\psi(\theta) = 30*(cos(\theta) - m)$

可以看作是对角度距离的优化,相比于余弦距离来说,当角度接近0和$\pi$时,余弦值会更密集.因此推测优化角度距离比优化余弦距离更有效果.



## 参考

[一文看懂softmax loss](https://blog.csdn.net/weixin_48958956/article/details/136812044)

[理解L-Softmax、A-Softmax 和 AM-Softmax](https://zhuanlan.zhihu.com/p/633337212)

[AM-Softmax](https://zhuanlan.zhihu.com/p/85252097)

[happynear技术专栏](https://zhuanlan.zhihu.com/c_1058780590651871232)

[Softmax理解之margin](https://zhuanlan.zhihu.com/p/52108088)

[从最优化的角度看待Softmax损失函数](https://zhuanlan.zhihu.com/p/45014864)

[Additive Margin Softmax for Face Verification](https://arxiv.org/pdf/1801.05599)