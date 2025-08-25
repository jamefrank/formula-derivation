# CircleLoss

## 简介



## 公式

### circle loss

$$
\begin{aligned}
L_{unified} &= log[1+\sum_{i=1}^K \sum_{j=1}^L exp(\gamma(s_n^j-s_p^i+m))] \\
&= log[1 + \sum_{j=1}^Lexp(\gamma(s_n^j+m)) \sum_{i=1}^Kexp(\gamma(-s_p^i))] \\
&\approx [log[\sum_{j=1}^Lexp(\gamma(s_n^j+m)) \sum_{i=1}^Kexp(\gamma(-s_p^i))]]_+ \\
&=[log\sum_{j=1}^Lexp(\gamma(s_n^j+m)) + log\sum_{i=1}^Kexp(\gamma(-s_p^i))]_+ \\
&=\gamma[LSE({\pmb{s_n}})-NLSE(\pmb{s_p})+m]_+ \\
&\approx \gamma[max(s_n)-min(s_p)+m]_+ 
\end{aligned}
$$

最后的推导的文字表述为
$$
使同类别(positive)最小的相似度比非同类别(negtive)最大的相似度大m
$$


### 最大值最小值的光滑近似

$$
LSE(\pmb{X};\gamma) = {1 \over \gamma} log\sum_iexp(\gamma*x_i) \approx max(\pmb{X}) \\
NLSE(\pmb{X};\gamma) = -{1 \over \gamma} log\sum_iexp(-\gamma*x_i) \approx min(\pmb{X})
$$

其中$\pmb{X}$表示多元变量,上述是对求多元变量中的子分量的最大/最小值的光滑近似

例如
$$
Softplus(x) = log(1+e^x) \approx max(x,0) = [x]_+  \\
\pmb{X} = [0,x]
$$


## 参考

[Circle loss学习与分享](https://www.jianshu.com/p/c1784de03d5b)

[如何构造一个平滑的最大值函数](https://matrix67.com/blog/archives/2830)

[寻求一个光滑的最大值函数](https://kexue.fm/archives/3290)

[从最优化的角度看待Softmax损失函数](https://zhuanlan.zhihu.com/p/45014864)

[如何理解与看待在cvpr2020中提出的circle loss？](https://www.zhihu.com/question/382802283)