# 渲染管线中的坐标变换

​	这里整体梳理一下渲染管线中的坐标变换，总是和相机标定中的针孔模型的一些坐标变换步骤搞混，因此需要进行梳理。

## 模型坐标系

​	物体自身的局部坐标系，我理解比如什么CAD数模啊，他们往往存在一个自身的坐标系，是画图或者构建的时候产生的。

## 世界坐标系

​	这个就是通常理解的世界坐标系，比如房间里面建立的一个全局坐标系。

## ***模型变换***

​	通过模型变换矩阵将模型坐标系中的点转换到世界坐标系中

​	可以将模型变换矩阵叫做物体在世界坐标系中的位姿
$$
p_{world} = M_{model}*p_{model}
$$


## 视图坐标系

​	因为之间做相机标定，以为视图坐标系和相机坐标系是一个东西。但是视图坐标系和相机坐标系并不是一个东西。具体定义如下：

​	相机坐标系：光心为原点，采用右手定则中的右下前定义的坐标系。

​	**视图坐标系：光心为原点，采用右手定则中的右上后定义的坐标系。**

​	因此相机在相机坐标系中看向的是+Z方向。但是在视图坐标系中看向的是-Z方向。

## ***视图变换***

​	把场景中所有物体从**世界坐标系**转换到**视图坐标系**。中间使用到视图变换矩阵$M_{view}$
$$
p_{view} = M_{view} * p_{world}
$$

## 裁剪坐标系

​	我是第一次知道有这个统一的学术的名字，其实就是那个棱台经过“压扁”后的长方体（大模型说这种描述不标准，标准描述是“**齐次坐标下的标准裁剪体**”，但是我感觉不太好理解，所以延用这种错误的说法，毕竟确实是齐次坐标）。棱台空间的由来：通过近裁剪面和远裁剪面截取视椎体后形成的棱台空间。

​	是经过投影变换+坐标剪裁之后的点所在的坐标系。
$$
p_{clip} = M_{per}*p_{view} \\
−w≤x,y,z≤w
$$

## ***投影变换***

$$
p_{clip} = M_{per}*p_{view}
$$



## ***坐标剪裁***

$$
−w≤x,y,z≤w
$$



## NDC坐标系

​	**Normalized Device Coordinates**，这个就是常说的那个立方体，$NDC \in [-1,1]^3$

## ***透视除法***

$$
x_{ndc} = x_{clip} / w_{clip} \\
y_{ndc} = y_{clip} / w_{clip} \\
z_{ndc} = z_{clip} / w_{clip} \\
$$

## 屏幕坐标系

​	我经常和相机标定中的图像坐标系（像素坐标系）搞混，这里详细梳理一下

​	像素坐标系：原点（图像左上角）+ x轴（向右） + y轴  （向下）

​	屏幕坐标系：DirectX/Vulkan/现代 OpenGL : 原点（左上角）+ x轴（向右） + y轴  （向下） ; 传统的OpenGL : 原点（左下角）+ x轴（向右） + y轴  （向上）和API相关

## ***视口变换***

​	将NDC立方体拉伸后回到像素平面上的操作
$$
\begin{aligned}
p_{screen space} &= M_{viewport}*p_{ndc} \\
&= 
\begin{bmatrix}
{w \over 2} && 0 && 0 && {w \over 2} \\
0 && {h \over 2} && 0 && {h \over 2} \\
0 && 0 && 1 && 0 \\
0 && 0 && 0 && 1
\end{bmatrix} *p_{ndc} \\
\end{aligned}
$$




![image-20251212182731418](/home/frank/.config/Typora/typora-user-images/image-20251212182731418.png)



# 投影变换的公式推导

## 正交投影  

games001

1. 采用右手系  $f < n < 0$
2. 映射立方体的关系 $(l,b,f):(-1,-1,-1); (r,t,n):(1,1,1)$

$$
-1 = scale_x * l + offset_x    \\
1 = scale_x * r + offset_x    \\

scale_x = 2 / (r - l)  \\
\begin{aligned}
offset_x &= 1 - scale_x * l  \\
&= 1-2l/(r-l) \\
&= (r-l-2l)/(r-l) \\
&= - (r+l)/(r-l) \\
&= -{{r+l} \over {r-l}} 
\end{aligned}
$$

同理可以推导出
$$
scale_y = 2/(t-b) \\
offset_y = - {{t+b} \over {t-b}} \\
scale_z = 2/(n-f) \\
offset_z = - {{n+f} \over {n-f}} \\
$$
最终可以得到正交投影矩阵
$$
M_{orth} = 
\begin{bmatrix}
{2 \over r-l} && 0 && 0 && -{{r+l} \over {r-l}} \\
0 && {2 \over t-b} && 0 && -{{t+b} \over {t-b}} \\
0 && 0 && {2 \over n-f} && -{{n+f} \over {n-f}} \\
0 && 0 && 0 && 1
\end{bmatrix}
$$

## 透视投影

games001 给出的证明思路是 将透视投影拆解成两步

1. 将视椎（棱台）压缩成长方体  $P$
2. 正交投影变换  $M_{orth}$

此时存在透视变换矩阵为$M_{per} = M_{orth}*P$

此时我们只需要推导出来$P$就可以得到 $M_{per}$了

这里详细拆解步骤一中的条件：

1. 近平面的所有点（例如$(l,b,n)$）的坐标保持不变
2. 其余点$(x,y)$的值根据$z$值进行线性缩放;  远平面上的$z$值仍然保证为 $f$​

现在来推导一下$P$矩阵是怎么来的？可能不严谨，但是很好奇是怎么得出来的，大家都说是构造出来的，又很模糊。

![img](https://pic3.zhimg.com/v2-0f8c759c9c44975c7aa1f28fd8b3dfac_1440w.jpg)

结合这张图来建立数学问题如下：

将视椎（棱台）内一点$(x,y,z)$  通过 转换矩阵 $P$  转换为坐标 $(x',y',z)$

写成矩阵形式为
$$
\begin{aligned}
\begin{bmatrix}
x' \\
y' \\
z \\
1
\end{bmatrix}
 = 
\begin{bmatrix}
? && ? && ? && ? \\
? && ? && ? && ? \\
? && ? && ? && ? \\
? && ? && ? && ? \\
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
z \\
1
\end{bmatrix}
\end{aligned}
$$
其中更具相似三角形可知
$$
x' = {nx \over z} \\
y' = {ny \over z} \\
$$
那么要求解的矩阵转化为
$$
\begin{aligned}
\begin{bmatrix}
{nx \over z} \\
{ny \over z} \\
z \\
1
\end{bmatrix}
&= 
\begin{bmatrix}
? && ? && ? && ? \\
? && ? && ? && ? \\
? && ? && ? && ? \\
? && ? && ? && ? \\
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
z \\
1
\end{bmatrix}
\end{aligned}
$$
进一步两边同时乘以$z$​转为如下

这里有个bug:为什么右边没变化呢，我是这么理解的，放进？里面去了，也不能不接受吧！！！

为什么能放进？里面去，因为右侧的这个向量我理解就是基分量，你有z的话，应该是线性的，不能出现  $z^2$​这种（这属于强行解释了）

1. 现在可以开始构造了，先构造$P$​的前两行

2. 然后构造第四行

3. 根据near+far平面的深度不变，可以构造两个等式$n^2=Ax+By+Cn+D$,$f^2=Ax+By+Cf+D$。（这里有个问题，为什么选用这两个面的深度构建等式？ 我的理解是视椎（棱台）就是根据这两个超参数确定的，视椎虽然有其他深度的平面，但是没有确定的参数）

   这里确定$ABCD$,其中上述两个等式指的是对平面内的所有点都成立，因此可以知道与$x,y$线性无关，因此$A=0,B=0$;剩下的  $CD$可以联立方程求解

$$
\begin{aligned}
\begin{bmatrix}
{nx} \\
{ny } \\
z^2 \\
z
\end{bmatrix}
&= 
\begin{bmatrix}
? && ? && ? && ? \\
? && ? && ? && ? \\
? && ? && ? && ? \\
? && ? && ? && ? \\
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
z \\
1
\end{bmatrix} \\

&= 
\begin{bmatrix}
n && 0 && 0 && 0 \\
0 && n && 0 && 0 \\
? && ? && ? && ? \\
? && ? && ? && ? \\
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
z \\
1
\end{bmatrix} \\

&= 
\begin{bmatrix}
n && 0 && 0 && 0 \\
0 && n && 0 && 0 \\
? && ? && ? && ? \\
0 && 0 && 1 && 0 \\
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
z \\
1
\end{bmatrix} \\

&= 
\begin{bmatrix}
n && 0 && 0 && 0 \\
0 && n && 0 && 0 \\
0 && 0 && n+f && -nf \\
0 && 0 && 1 && 0 \\
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
z \\
1
\end{bmatrix} \\
\end{aligned}
$$



$P$矩阵已知后，就知道了透视投影矩阵了，同时要注意的是，每次对某个点做完透视投影后，还要归一化才行，这一步也对上了之前求解$P$的时候莫名奇妙的左右两侧乘以了一个$z$

**莫名奇妙的等式左右同时乘以$z$​的终极解释**

现在的理解是，$z$的乘除其实是一步坐标系转换的操作，是归一化坐标系和相机坐标系之间的转换，可以参考相机内外参标定来理解。

### 参考资料

https://zhuanlan.zhihu.com/p/122411512

https://www.bilibili.com/video/BV1ZY41157TR/?spm_id_from=333.337.search-card.all.click&vd_source=bdda56b3baf0c6d4d1d6028820df7bcc



# 光栅化

