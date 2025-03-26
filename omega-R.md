# 角速度与旋转矩阵的关系

记录一下 角速度 $\vec{\omega}$ 与 旋转矩阵 $R$ 之间的关系的推导过程

## 质点线速度与角速度的关系

$$
\vec{v} = \vec{\omega} \times \vec{r}
$$

其中 $\vec{v}$ 为质点线速度，$\vec{\omega}$ 为角速度，$\vec{r}$ 为质点在直角坐标系下的矢径

## 旋转矩阵

$$
R_{ib} = \begin{bmatrix}
\vec{x_{ib}} & \vec{y_{ib}} & \vec{z_{ib}}
\end{bmatrix} 
$$

其中 $i$ 表示全局坐标系，$b$表示局部坐标系，$\vec{x_{ib}}$、$\vec{y_{ib}}$、$\vec{z_{ib}}$ 为旋转矩阵的列向量，分别是$b$坐标系的x轴，y轴，z轴的单位向量在全局坐标系下的坐标


## 推导过程1（全局坐标系角速度与旋转矩阵的关系）
$$
\begin{aligned}
\dot{R_{ib}} &= \begin{bmatrix}
\dot{\vec{x_{ib}}} & \dot{\vec{y_{ib}}} & \dot{\vec{z_{ib}}}
\end{bmatrix}  \\
&= \begin{bmatrix}
\vec{\omega_i} \times \vec{x_{ib}} & \vec{\omega_i} \times \vec{y_{ib}} & \vec{\omega_i} \times \vec{z_{ib}}
\end{bmatrix} \\
&= \begin{bmatrix}
S(\vec{\omega_i}) \cdot \vec{x_{ib}} & S(\vec{\omega_i}) \cdot \vec{y_{ib}} & S(\vec{\omega_i}) \cdot \vec{z_{ib}}
\end{bmatrix} \\
&= S(\vec{\omega_i}) \cdot R_{ib}
\end{aligned}
$$

注意 $\vec{\omega_i}$ 为刚体在全局坐标系下的角速度
但是更多的时候，已知的是传感器上获取到的角速度，这些传感器固连在运动物体上，因此获取局部坐标系的角速度和旋转矩阵之间的关系会更有意义。

## 推导过程2（局部坐标系角速度与旋转矩阵的关系）

### 角速度变换原理
$$
\vec{\omega_i} = R_{ib} \cdot \vec{\omega_b}
$$

### 叉乘分配率
两个向量的相对位置关系在两个向量经过相同旋转后，不会发生改变
$$
R_{ib}\vec{p_b} \times R_{ib}\vec{q_b} = R_{ib}(\vec{p_b} \times \vec{q_b})
$$

### 反对称运算分配率
$$
(R \cdot \vec{p})^{\land} =  R \cdot \vec{p}^{\land} \cdot R^{T}
$$
#### 证明
$$
\begin{aligned}
(R \cdot \vec{p})^{\land} \cdot \vec{v} &= (R \cdot \vec{p}) \times \vec{v} \\
&= R \cdot R^T \cdot [(R \cdot \vec{p}) \times \vec{v}] \\
&= [R \cdot R^T \cdot (R \cdot \vec{p})] \times [R \cdot R^T \cdot \vec(v)] \\
&= R \cdot ( [R^T \cdot R \cdot \vec{p}] \times [R^T \cdot \vec{v}]) \\
&= R \cdot (\vec{p} \times (R^T \cdot \vec{v})) \\ 
&= R \cdot \vec{p}^\land \cdot R^T \cdot \vec{v}
\end{aligned}
$$

### 推导过程
$$
\begin{aligned}
\dot{R_{ib}} &= S(\vec{\omega_i}) \cdot R_{ib} \\
&= S(R_{ib} \cdot \vec{\omega_b}) \cdot R_{ib} \\
&= R_{ib} \cdot S(\vec{\omega_b}) \cdot R_{ib}^T \cdot R_{ib} \\
&= R_{ib} \cdot S(\vec{\omega_b})
\end{aligned}
$$

# 参考资料
[参考资料](https://zhuanlan.zhihu.com/p/681296457)