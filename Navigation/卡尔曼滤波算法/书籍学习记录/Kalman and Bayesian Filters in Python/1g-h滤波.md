学习来源：
https://github.com/rlabbe/Kalman-and-Bayesian-Filters-in-Python
作者：
Roger Labbe（github）


# 第一章：g-h-filter

小知识点：

1：plant 有系统的意思，这词来自控制系统理论

2：rings现象：是指信号以正弦类型的模式上冲和下冲数据

![IMAGE ALT TEXT HERE](https://github.com/xdwgood/Navigation-and-control/blob/xdwgood-patch-1/111.png)
上图中我们假设A和B表示俩种传感器对同一状态进行测量所得到的结果，其中A的精度较高B较低，这个图直观的告诉我们：

我们在进行状态估计的时候不应该放弃任何相关信息,即使它可能很糟糕！!(帮助我们缩小估计错误)

### 什么是g-h滤波?

g-h滤波通过调整我们对某一状态的测量值和预测值的比例，来估计出当前系统这一状态的最优值

参数g：调整我们更偏向测量值还是预测值，g越大我们越相信测量值

参数h：调整系统响应速度（h越大，系统对变化的信号反映越迅速，越倾向测量），调整系统倾向测量变化速度还是预测的（先验）变化速度

一点思考：本质上g-h滤波也在试图解决一个矛盾：系统快速的响应和稳定状态下的保持？

### python 代码实现：

data:要滤波的数据

x0、dx：初始状态和变换速度

```
def g_h_filter(data, x0, dx, g, h, dt):
    x_est = x0
    results = []
    for z in data:
        # prediction step
        x_pred = x_est + (dx*dt)
        dx = dx

        # update step
        residual = z - x_pred
        dx = dx + h * (residual) / dt
        x_est = x_pred + g * residual
        results.append(x_est)
    return np.array(results)
```