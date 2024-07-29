# 算平均

从学会数数起，笔者就在学习如何计算平均数，随着知识的增长，笔者发现计算平均数的方法不仅仅只有小学时候的简单算法。于是在这里记录一下常用的算平均数，或者平滑的方法，有些符号滥用，权当潦草笔记写一写。


## MA

Moving Average，移动平均，最基础的平均值，就是设定一个滑动窗口，然后计算这个滑动窗口内元素的均值，可以加权也可以不加权，用离散卷积很容易实现：
$$
\begin{aligned}
h(t) &= \begin{cases}  
w_t & \text{if } 0 \leq t < N \\  
0 & \text{otherwise}  
\end{cases} \\
\\
y(t) = (x &* h)(t) = \sum_{i=-\infty}^{\infty} x(i) h(t-i)y(t)

​\end{aligned}
$$
当然要注意如果数据数目小于窗口数目$N$的时候需要特别关照一下（从$N$开始计算？填充一下？动态窗口？）。  

## CMA

Cumulative Moving Average，累计移动平均，强化学习里面经常用来累计算期望，而且默认不加权，公式为:
$$
	\bar X_t = \frac{t-1}{t}\bar X_{t-1} + \frac{1}{t}x_t
$$
简单说明就是：
$$
\begin{aligned}
\bar X_t &= \frac{1}{t}\sum_{i=1}^tx_i\\
&=\frac{t-1}{t}[\frac{1}{t-1}\sum_{i=0}^{t-1}x_i + \frac{1}{t-1}x_t]\\
&=\frac{t-1}{t}\bar X_{t-1} + \frac{1}{t}x_t
\end{aligned}
$$

## EMA

Exponential Moving Average，指数移动平均，最常用的算法，有时候模型的参数的更新（rl，ss），或者是算平滑曲线（tensorboard）都经常用这个方法。公式为：
$$
	\bar X_t = \alpha\bar X_{t-1} + (1-\alpha)x_t \quad s.t. \quad 0 \leq \alpha \leq1
$$
通常$\alpha=\frac{2}{N-1}$， 这里的$N$是窗口大小，笔者不理解为什么这样用，不no matter，一般用的时候也只是调整$\alpha$就可以了。而为什么叫做指数移动平均呢？笔者认为这是一个递推的式子，其中把这个$\bar X_t$式子展开，就会发现对于第$n$时刻的数据点$X_n$， 它的系数是$(1-\alpha)^{t+1-n}$，就是越接近$t$时刻的数据点权重越大，这个权重是一个关于$t$的指数函数。