学习来源：
https://github.com/rlabbe/Kalman-and-Bayesian-Filters-in-Python
作者：
Roger Labbe（github）


# 第十章：无痕卡尔曼滤波器（UKF）

小知识点：

1：蒙特卡洛方法（Monte Carlo）：每次状态更新我们生成500,000（很多）点（这个点包括当前状态和概率），将它们传递给函数，然后计算结果的均值和方差。粒子滤波器使用该方法。

2：UKF中使用函数来表达F和H，而不是矩阵，因为矩阵就意味着线性代数

3:RTS平滑算法：是根据后续数据判断优化当前状态，和kf相反，kf是用过去的状态估计未来状态

### Sigma点-----分布采样

Sigma点定义： 每一个点代表当前状态值和相应信念（包括当状态值、均值权重、方差权重）

##### Van der Merwe's 缩放生成sigma点算法（2004年）

第0个sigma点X0 = u------W0（mean） = 𝜆 / n + 𝜆------W0（c） = 𝜆/𝑛+𝜆 + 1 - 𝛼2 + 𝛽

其它sigma点计算如下，给定公式： 𝜆=𝛼2(𝑛+𝜅)−𝑛，计算如下：

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/137.png)

其中 Σ=𝐒𝐒𝖳，𝐒=cholesky(𝐏)，下标i，进行选择，代码实现如下：

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/139.png)

其它sigma均值和权重计算如下图：

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/138.png)

建议的合理选择：

𝛽 = 2; 𝜅 = 3 − 𝑛; 0 ≤ 𝛼 ≤ 1(𝛼越大采样越分散); n代表多少维状态（eg：x方向速度和位置，即 n = 2）

### UKF算法实现

##### 预测步骤

1：使用特殊函数生成多个sigma点和相应均值权重+方差权重（该函数输入仅为：x和P）

2：把每一个点放入我们定义好的过程传递函数中，得到先验sigma点

3：使用UT传输函数，将多个先验sigma点进行计算（权重求和），得到最终预测的x和P，全部过程如下图所示（其中y是步骤2得到的点的集合）：

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/135.png)

##### 更新步骤

1：把预测步骤中生成的所有sigma点y，传入测量函数转移到测量空间

2：使用UT传输进行均值和方差计算

3：计算残差，计算状态变量和测量值的协方差

4：计算卡尔曼增益，应用卡尔曼增益得到后验状态和协方差矩阵（P），更新过程如下图所示：

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/136.png)

### 使用UKF（预测机器人位置）

前提：机器人上的传感器提供距离和方位，由于我们需要使用三角函数和勾股定理，因此测量是非线性的。同时汽车的运动过程也是非线性的。

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/143.png)

设计运动方程如下图：其中x/y为位置，𝜃为机器人航向,即过程传递函数如下（其中𝛽和R为测量传感器提供）

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/140.png)

设计状态变量为 x    y   𝜃，设计u = [𝑣 𝛼]，分别控制速度和汽车转向角

##### 设计系统模型

我们将系统建模为非线性运动模型加白噪声，**Note**这里的𝛽计算来自下图，而不是测量传感器提供的，预测环节与传感器测量无关

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/142.png)


![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/141.png)

##### 设计测量模型

设计测量传递模型如下

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/145.png)

##### 设计测量噪声如下

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/144.png)

......未完待续（目前理解的还不够深）