# imu测量模型

## 陀螺仪测量模型

$$
\tilde{\omega}_b(t) = \omega_b(t) + b_g(t) + \eta_g(t)
$$

$\tilde{\omega}_b(t)$ ：imu在$t$时刻对载体坐标系$b$的角速度在载体坐标系$b$下的测量值

$\omega_b(t)$ ：imu在$t$时刻对载体坐标系$b$的角速度在载体坐标系$b$下的真实值

$b_g(t)$ ： $t$时刻陀螺仪的bias （确定性误差）

$\eta_g(t)$ ：$t$时刻陀螺仪输出中含有的白噪声 （随机性误差）

## 加速度计测量模型

$$
\begin{aligned}
\tilde{a}_b(t) &= R_{bi}(a_i(t)-g_i)+b_a(t)+\eta_a(t) \\
&= R_{ib}^T(a_i(t)-g_i)+b_a(t)+\eta_a(t)
\end{aligned}
$$

$R_{ib}$ ：当前时刻载体/imu在世界坐标系$i$下的姿态

$\tilde{a}_b(t)$ ：imu在$t$时刻对载体坐标系$b$的加速度在载体坐标系$b$下的测量值

$a_i(t)$ ：imu在$t$时刻对载体坐标系$b$的加速度在世界坐标系$i$​下的真实值

$g_i$ ：世界坐标系$i$下的重力加速度

$b_a(t)$ ：在$t$时刻加速度计的bias（确定性误差）

$\eta_a(t)$ ：$t$时刻加速度计输出中含有的白噪声 （随机性误差）

# 运动学模型

## 假设

+ 忽略地球自转
+ 运行区域水平面是平面
+ 重力矢量固定

## 模型

$$
\dot{R}_{ib} = R_{ib}[\omega_b]_{\times} \\
\dot{v}_i = a_i \\
\dot{p}_i = v_i
$$

离散形式如下
$$
R_{ib}(t+\delta t) = R_{ib}(t)Exp(\omega_b(t) \cdot \delta t) \\
\\
v_i(t+ \delta t) = v_i(t) + a_i(t) \cdot \delta t \\
\\
p_i(t+ \delta t) = p_i(t) + v_i(t) \cdot \delta t + {1 \over 2}a_i(t)\cdot {\delta t}^2
$$
这里需要解释一下对应的指数运算和对数运算

$Exp: R^3 \rightarrow SO(3)$ ：旋转向量 $\rightarrow$ $3\times3$正交阵

$Log:SO(3) \rightarrow R^3$ ：$3\times3$正交阵 $\rightarrow$ 旋转向量

$exp:so(3) \rightarrow SO(3)$ ：$3\times3$反对称阵 $\rightarrow$  $3\times3$正交阵

$log: SO(3) \rightarrow so(3)$ ：$3\times3$正交阵 $\rightarrow$ $3\times3$反对称阵



将测量模型带入离散形式的运动方程可得：
$$
R_{ib}(t+\delta t) = R_{ib}(t) \cdot Exp[(\tilde{\omega}_b(t) - b_g(t) - \eta_{gd}(t)) \cdot \delta t] \\
\\
\begin{aligned}
v_i(t+\delta t) = v_i(t) + [R_{ib}*(\tilde{a}_b(t)-b_a(t)-\eta_{ad}(t)) + g_i] \cdot \delta t
\end{aligned} \\
\\

\begin{aligned}
p_i(t+\delta t) &= p_i(t) + v_i(t) \cdot \delta t + {1 \over 2}a_i(t)\cdot {\delta t}^2 \\
&= p_i(t) + v_i(t) \cdot \delta t + {1 \over 2}[R_{ib}*(\tilde{a}_b(t)-b_a(t)-\eta_{ad}(t)) + g_i] \cdot {\delta t}^2 \\
&= p_i(t) + v_i(t) \cdot \delta t + {1 \over 2}g_i \cdot {\delta t}^2 + {1 \over 2}[R_{ib}*(\tilde{a}_b(t)-b_a(t)-\eta_{ad}(t))]  \cdot {\delta t}^2 \\
\end{aligned}
$$
其中离散(discrete)的噪声项和连续的噪声项之间的协方差存在如下关系：
$$
Cov(\eta_{gd}(t)) = {1 \over \delta t} Cov(\eta_g(t)) \\
Cov(\eta_{ad}(t)) = {1 \over \delta t} Cov(\eta_a(t)) \\
$$
假设$\delta t$恒定，采样频率固定不变，同时每个离散时刻由$k=0,1,2,...$表示，离散运动方程表示如下：
$$
R_{k+1}^{ib} = R_{k}^{ib} \cdot Exp[(\tilde{\omega}_k^b - b_k^g - \eta_k^{gd}) \cdot \delta t] \\
v_{k+1}^i = v_k^i + [R_k^{ib}*(\tilde{a}_k^b-b_k^a-\eta_k^{ad}) + g^i] \cdot \delta t \\
p_{k+1}^i = p_k^i + v_k^i \cdot \delta t + {1 \over 2}g_i \cdot {\delta t}^2 + {1 \over 2}[R_k^{ib}*(\tilde{a}_k^b-b_k^a-\eta_k^{ad})]  \cdot {\delta t}^2 \\
$$

## 累积测量

​	使用$k=i,i+1,...,j-1$时刻的所有的imu测量以及$k=i$时刻的状态值$R_{k=i}^{ib},v_{k=i}^i,p_{k=i}^i$根据运动模型计算得到$k=j$时刻的状态值$R_{k=j}^{ib},v_{k=j}^i,p_{k=j}^i$，具体计算如下：
$$
R_{k=j}^{ib} = R_{k=i}^{ib} \cdot \prod_{k=i}^{j-1} Exp[(\tilde{\omega}_k^b - b_k^g - \eta_k^{gd}) \cdot \delta t] \\
v_{k=j}^i = v_{k=i}^i + \sum_{k=i}^{j-1}[R_k^{ib}*(\tilde{a}_k^b-b_k^a-\eta_k^{ad} + g^i)] \cdot \delta t \\
p_{k=j}^i = p_{k=i}^i + \sum_{k=i}^{j-1}v_k^i \cdot \delta t + {j-i \over 2}g_i \cdot {\delta t}^2 + {1 \over 2}\sum_{k=i}^{j-1}[R_k^{ib}*(\tilde{a}_k^b-b_k^a-\eta_k^{ad})]  \cdot {\delta t}^2 \\
$$

## 预积分

​	原因：如果$R_{k=i}^{ib},v_{k=i}^i,p_{k=i}^i$发生变化后，那么根据上述运动方程需要重新积分计算$R_{k=j}^{ib},v_{k=j}^i,p_{k=j}^i$，这样就会重复计算，因此引出了预积分的概念。
$$
\begin{aligned}
\delta R_{ij}^{ib} &= R_{k=i}^TR_{k=j} \\
&= \prod_{k=i}^{j-1} Exp[(\tilde{\omega}_k^b - b_k^g - \eta_k^{gd}) \cdot \delta t] \\
\end{aligned}
$$

$$
\begin{aligned}
\delta v_{ij} &= R_i^T(v_j-v_i-g \cdot \delta t_{ij}) \\
&= \sum_{k=i}^{j-1}[\delta R_{ik}^{ib}*(\tilde{a}_k^b-b_k^a-\eta_k^{ad})] \cdot \delta t 
\end{aligned}
$$

$$
\begin{aligned}
\delta p_{ij} &= R_i^T([p_j-p_i]-v_i \cdot \delta t_{ij} - {1 \over 2}g \cdot \delta t_{ij}^2) \\
&=  R_i^T([\sum_{k=i}^{j-1}(v_k \delta t + {1 \over 2}g \cdot \delta t^2 + {1 \over 2}R_k\xi_k\cdot \delta t^2)] - \sum_{k=i}^{j-1}v_i \cdot \delta t - {(j-i)^2 \over 2}g \cdot \delta t^2) \\
&= R_i^T[\sum_{k=i}^{j-1}(v_k-v_i)\delta t + ({j-i \over 2} - {(j-i)^2 \over 2})g \cdot \delta t^2 + {1 \over 2}\sum_{k=i}^{j-1}R_k\xi_k\delta t^2] \\
&= R_i^T[\sum_{k=i}^{j-1}(v_k-v_i)\delta t + \sum_{k=i}^{j-1}(k-i)g \cdot \delta t^2 + {1 \over 2}\sum_{k=i}^{j-1}R_k\xi_k\delta t^2] \\
&= R_i^T \sum_{k=i}^{j-1}\{ [v_k-v_i-(k-i)g\cdot \delta t] \delta t +  {1 \over 2} R_k\xi_k\delta t^2\} \\
&= R_i^T \sum_{k=i}^{j-1}[(v_k-v_i-g\cdot \delta t_{ik}) \delta t + {1 \over 2} R_k\xi_k\delta t^2] \\
&= R_i^T \sum_{k=i}^{j-1}[R_i \cdot \delta v_{ik} \cdot \delta t + {1 \over 2} R_k\xi_k\delta t^2] \\
&= \sum_{k=i}^{j-1}[\delta v_{ik} \cdot \delta t + {1 \over 2} \delta R_{ik}\xi_k\delta t^2] \\
&= \sum_{k=i}^{j-1}[\delta v_{ik} \cdot \delta t + {1 \over 2} \delta R_{ik}(\tilde{a}_k^b-b_k^a-\eta_k^{ad})\delta t^2] \\
\end{aligned}
$$

## 理想值“+”白噪声

​	假设预积分区间内的bias相同
$$
b_i^g=b_{i+1}^g=...=b_j^g \\
b_i^a=b_{i+1}^a=...=b_j^a
$$
​	李群流行公式
$$
Exp(\vec{\phi}+\delta \vec{\phi}) \overset{(1)}{\approx} Exp(\vec{\phi}) \cdot Exp(J_r(\vec{\phi}) \cdot \delta \vec{\phi}) \\
Log[Exp(\vec{\phi}) \cdot Exp(\delta \vec{\phi})] = \vec{\phi} + J_r^{-1}(\vec{\phi}) \cdot \delta \vec{\phi} \\
\\
J_r(\vec{\phi}) = I-{1-cos(||\vec{\phi}||) \over ||\vec{\phi}||^2}\vec{\phi}_{\times} + {||\vec{\phi}||-sin(||\vec{\phi}||) \over ||\vec{\phi}||^3}(\vec{\phi}_{\times})^2 \\
J_r^{-1}(\vec{\phi}) = I+{1 \over 2} \vec{\phi}_{\times} + ({1 \over ||\vec{\phi}||^2} - {1+cos(||\vec{\phi}||) \over 2 \cdot ||\vec{\phi}|| \cdot sin(||\vec{\phi}||)})(\vec{\phi}_{\times})^2 \\

\\

R \cdot Exp(\vec{\phi}) \cdot R^T = exp(R \vec{\phi}_{\times}R^T)=Exp(R\vec{\phi}) \\
\rightarrow Exp(\vec{\phi}) \cdot R \overset{(2)}{=} R \cdot Exp(R^T \vec{\phi})
$$

### 旋转预积分 噪声分离

$$
\begin{aligned}
\delta R_{ij} &= \prod_{k=i}^{j-1}Exp[(\tilde \omega_k-b_i^g-\eta_k^{gd}) \cdot \delta t] \\
&= \prod_{k=i}^{j-1}Exp[(\tilde \omega_k-b_i^g)\delta t - \eta_k^{gd}\delta t] \\
&\overset{(1)}{\approx} \prod_{k=i}^{j-1} \{ Exp[(\tilde \omega_k-b_i^g)\delta t] \cdot Exp[-J_r((\tilde \omega_k-b_i^g)\delta t) \cdot \eta_k^{gd} \delta t] \} \\
&= \prod_{k=i}^{j-1}Exp(D_k)Exp(M_k) \\
&= Exp(D_i)Exp(M_i)Exp(D_{i+1})Exp(M_{i+1})...Exp(D_{j-2})Exp(M_{j-2})Exp(D_{j-1})Exp(M_{j-1}) \\
&= Exp(D_i) \cdot Exp(M_i)Exp(D_{i+1}) \cdot Exp(M_{i+1})...Exp(D_{j-2}) \cdot Exp(M_{j-2})Exp(D_{j-1}) \cdot Exp(M_{j-1}) \\
&\overset{(2)}{=} Exp(D_i) \cdot Exp(D_{i+1})Exp[Exp^T(D_{i+1})M_i] \cdot Exp(D_{i+2})Exp[Exp^T(D_{i+2})M_{i+1}] ...
Exp(D_{j-2})Exp[Exp^T(D_{j-2})M_{j-3}] \cdot Exp(D_{j-1})Exp[Exp^T(D_{j-1})M_{j-2}] \cdot Exp(M_{j-1}) \\
&\overset{(2)}{=} Exp(D_i)Exp(D_{i+1}) \cdot Exp[Exp^T(D_{i+1})M_i]Exp(D_{i+2}) \cdot Exp[Exp^T(D_{i+2})M_{i+1}]Exp(D_{i+3}) ...
Exp[Exp^T(D_{j-2})M_{j-3}]Exp(D_{j-1})  \cdot Exp[Exp^T(D_{j-1})M_{j-2}] \cdot Exp(M_{j-1}) \\
&\overset{(2)}{=} Exp(D_i)Exp(D_{i+1})Exp(D_{i+2}) \cdot Exp\{[Exp(D_{i+1})Exp(D_{i+2})]^TM_i\} Exp(D_{i+3}) \cdot Exp\{[Exp(D_{i+2})Exp(D_{i+3})]^TM_{i+1}\} Exp(D_{i+4})... Exp\{[Exp(D_{j-2})Exp(D_{j-1})]^TM_{j-3}\}  \cdot Exp[Exp^T(D_{j-1})M_{j-2}] \cdot Exp(M_{j-1}) \\
&\overset{(2)}{=}  ... \\
&\overset{(2)}{=}  Exp(D_i)Exp(D_{i+1})Exp(D_{i+2})...Exp(D_{j-1}) \cdot Exp\{[Exp(D_{i+1})Exp(D_{i+2})...Exp(D_{j-1})]^TM_i\} \cdot Exp\{[Exp(D_{i+2})Exp(D_{i+3})...Exp(D_{j-1})]^TM_{i+1}\} ... Exp[Exp^T(D_{j-1})M_{j-2}] \cdot Exp(M_{j-1}) \\
&= \prod_{k=i}^{j-1}Exp(D_i) \cdot \prod_{k=i}^{j-2}Exp[(\prod_{n=k+1}^{j-1}D_n)^TM_k] \cdot Exp(M_{j-1}) \\
&= \delta \tilde{R}_{ij} \cdot \prod_{k=i}^{j-1}Exp(\delta \tilde{R}_{k+1,j}^T \cdot M_k) \\
&= \delta \tilde{R}_{ij} \cdot Exp(-\delta \vec{\phi}_{ij})
\end{aligned}
$$

其中定义如下变量的等价形式：
$$
D_k = (\tilde{\omega}_k-b_i^g) \delta t \\
M_k = -J_r(D_k) \cdot \eta_k^{gd} \delta t \\
\delta \tilde{R}_{jj} = I \\
\delta \tilde{R}_{ij} = \prod_{k=i}^{j-1}Exp(D_k) \\
Exp(-\delta \vec{\phi}_{ij}) = \prod_{k=i}^{j-1}Exp(\delta \tilde{R}_{k+1,j}^T \cdot M_k)
$$
其中 认为 $\delta \tilde{R}_{ij}$为旋转预积分的测量值，$Exp(-\delta \vec{\phi}_{ij})$ 为其测量噪声

### 速度预积分 噪声分离

​	流行相关公式
$$
Exp(\vec{\phi}) \overset{(1)} {\approx} I + [\vec{\phi}]_{\times}  ,\text {when} \vec{\phi} \approx 0 \\
[a]_{\times} \cdot b \overset{(2)}{=} - [b]_{\times}\cdot a
$$


​	将旋转预积分 噪声分离公式带入到速度预积分公式中
$$
\begin{aligned}
\delta v_{ij} &= \sum_{k=i}^{j-1}[\delta R_{ik}*(\tilde{a}_k^b-b_k^a-\eta_k^{ad})] \cdot \delta t \\
&\approx \sum_{k=i}^{j-1}[ \delta \tilde{R}_{ik} \cdot Exp(-\delta \vec{\phi}_{ik}) \cdot (\tilde{a}_k^b-b_k^a-\eta_k^{ad})] \cdot \delta t \\
& \overset{(1)}{\approx} \sum_{k=i}^{j-1}  \delta \tilde{R}_{ik} \cdot (I-[\delta \vec{\phi_{ik}}]_{\times}) \cdot (\tilde{a}_k^b-b_k^a-\eta_k^{ad}) \cdot \delta t \\
&\approx \sum_{k=i}^{j-1} \delta \tilde{R}_{ik} \cdot(I-[\delta \vec{\phi_{ik}}]_{\times}) \cdot (\tilde{a}_k^b-b_k^a) \cdot \delta t -   \delta \tilde{R}_{ik} \eta_k^{ad} \delta t + \cancel{\delta \tilde{R}_{ik}[\delta \vec{\phi_{ik}}]_{\times}\eta_k^{ad} \delta t} \\
&= \sum_{k=i}^{j-1} \delta \tilde{R}_{ik}(\tilde{a}_k^b-b_k^a) \cdot \delta t - \delta \tilde{R}_{ik}[\delta \vec{\phi_{ik}}]_{\times}(\tilde{a}_k^b-b_k^a) \cdot \delta t -   \delta \tilde{R}_{ik} \eta_k^{ad} \delta t \\
&\overset{(2)}{=} \sum_{k=i}^{j-1}\delta \tilde{R}_{ik}(\tilde{a}_k^b-b_k^a) \cdot \delta t + \delta \tilde{R}_{ik}[\tilde{a}_k^b-b_k^a]_{\times} \cdot \delta \vec{\phi} \cdot \delta t -   \delta \tilde{R}_{ik} \eta_k^{ad} \delta t \\
&= \sum_{k=i}^{j-1}[\delta \tilde{R}_{ik} \cdot (\tilde{a}_k^b-b_k^a) \cdot \delta t] + \sum_{k=i}^{j-1}[\delta \tilde{R}_{ik} \cdot [\tilde{a}_k^b-b_k^a]_{\times} \cdot \delta t - \delta \tilde{R}_{ik} \eta_k^{ad} \delta t] \\
&= \delta \tilde v_{ij} + \delta v_{ij}
\end{aligned}
$$
 其中认为$\delta \tilde v_{ij}$ 为速度增量预积分测量值，$\delta v_{ij}$为其测量噪声

### 位移预积分 噪声分离

