# Triplet Loss

## 简介

1. 来源: 最初是在 [FaceNet: A Unified Embedding for Face Recognition and Clustering](https://arxiv.org/abs/1503.03832) 论文中提出的
2. 和Softmax-CrossEntropyLoss的区别: SC是用于多分类任务,最终是为了获取类别.T是为了学习用来分类的embedding.

## 公式

$$
L = max(d(a,p)-d(a,n)+margin,0)
$$

其中:

- d(x,y)表示的是自定义的距离函数
- a表示锚点anchor,模型做特征提取后的特征向量
- p表示正例positive,模型做特征提取后的特征向量
- n表示负例negative,模型做特征提取后的特征向量
- margin是一个非负的边距参数,用于控制正例和负例之间的最小距离差
- max的含义是说,当$d(a,p)-d(a,n)+margin<0$的时候,模型已经较好的区分正负例了,不用更新模型

## 参考

[Triplet-Loss原理及其实现](http://lawlite.me/2018/10/16/Triplet-Loss%E5%8E%9F%E7%90%86%E5%8F%8A%E5%85%B6%E5%AE%9E%E7%8E%B0/#3%E3%80%81%E8%AE%AD%E7%BB%83%E6%96%B9%E6%B3%95)

[Triplet Loss原理及实现](https://zhuanlan.zhihu.com/p/443560932)

[Triplet Loss解析及示例计算](https://blog.csdn.net/weixin_51524504/article/details/141393644)
