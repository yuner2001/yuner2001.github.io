---
created: 2024-01-28T22:35
updated: 2024-01-31T20:22
---

# 最后一个 Batch 不完整时怎么计算相关指标

在模型训练时，通常将数据按 batch 送入模型中进行计算。这样的话就会面临一个问题，总数据的个数不能被 batch size 整除时，最后一个 batch 的数据量往往不是完整的 batch size 大小。一般的做法可用将最后一个 batch 丢弃，或者就直接当作全 batch 来计算。但是随着 batch size 的增大，总的 batch 数目减少，最后一个 batch 的大小可能和实际的 batch size 相去甚远，可能会导致结果有较大误差；或者说在某些场景下，我们需要准确的数据，不能舍弃最后一个 batch，就应该对 loss 和各个指标的计算方法做出一定修改。下面就讨论下怎么计算才能得到一个相对精确的值。

首先做一些约定：假设有 $M$ 组（batch）数据,总量为$N$，每组前$M-1$组的数据量相同，batch size 均为$N_b$，其中第 $m$ 组的数据个数为 $N_m$ ，其中第 $i$ 个数据的某一指标记作为 $d_{mi}$ （loss、acc、iou 等之类的指标）， 则有：

$$
\begin{aligned}
&N_b = N_1 = N_2 = ... N_{M-1} \\
&\sum_{i=1}^MN_i = N \\
\end{aligned}
$$

按照每个 batch 求平均，然后再求 batch 平均的平均的方法，可得全局的该指标的均值为：

$$
\begin{aligned}
\bar D = \frac{\sum_{m}^{M}\frac{\sum_i^{N_m}d_{mi}}{N_m}}{M} =& \sum_{m}^{M}\frac{\sum_i^{N_m}d_{mi}}{N_mM} \\
=& \sum_m^{M-1}\frac{\sum_i^{N_m}d_{mi}}{N_mM} + \frac{\sum_i^{N_M}d_{Mi}}{N_MM} \\
=& \sum_m^{M-1}\frac{\sum_i^{N_m}d_{mi}}{N_bM} + \frac{\sum_i^{N_M}d_{Mi}}{N_MM} \\
=& \sum_m^{M-1}\frac{\sum_i^{N_m}d_{mi}}{N_bM} + \frac{\sum_i^{N_M}d_{Mi}}{N_MM}  \\
=& \frac{1}{M}[\sum_m^{M-1}\frac{\sum_i^{N_m}d_{mi}}{N_b} + \frac{\sum_i^{N_M}d_{Mi}}{N_M}]
\end{aligned}
$$

计算时候一把都是把中括号中的均值项全部算出来，然后最后除上 M。而真正的算法应该是：

$$
\begin{aligned}
\bar D_{gt} = \frac{1}{N}\sum_j^Nd_j &= \frac{1}{(M-1)N_b+N_M}[\sum_m^{M-1}\sum_i^{N_m}d_{mi} + \sum_i^{N_M}d_{Mi}] \\
&=\sum_m^{M-1}\frac{\sum_i^{N_m}d_{mi}}{(M-1)N_b+N_M} + \frac{\sum_i^{N_M}d_{Mi}}{(M-1)N_b+N_M} \\
&=\frac{1}{M}[\sum_m^{M-1}\frac{M\sum_i^{N_m}d_{mi}}{(M-1)N_b+N_M} + \frac{M\sum_i^{N_M}d_{Mi}}{(M-1)N_b+N_M}]
\end{aligned}
$$

简单观察一下项 $\sum_i^{N_m}d_{mi}$ 和项 $\sum_i^{N_M}d_{Mi}$ 的系数,其实只需要将前 $M-1$个全 batch 算出来均值$\sum_m^{M-1}\frac{\sum_i^{N_m}d_{mi}}{N_b}$乘系数 $\frac{MN_b}{(M-1)N_b+N_M}$ ，把最后一个不全的 batch 算出来的均值 $\frac{\sum_i^{N_M}d_{Mi}}{N_M}$乘上系数$\frac{MN_M}{(M-1)N_b+N_M}$,就可以按照原来的先求 batch 均值再求 batch 之间均值的算法取计算了。

最后来点代码验证一下, 从程序上说会有一些计算的误差，但这是由于计算机浮点计算的性质导致的：

```python
import random
# Generate random data
random_list = [random.randint(1, 100) for _ in range(23)]
data = random_list.copy()
print('data: ', data)

# Build Batches
batches, batch = [], []
for i in range(len(random_list)):
    batch.append(random_list.pop(0))
    if (i+1) % 5 == 0 or len(random_list) == 0:
        batches.append(batch.copy())
        batch = []
print('batches: ',batches)

def batch_avg(batches):
    avgs = []
    for batch in batches:
        assert not len(batch) == 0
        avgs.append(sum(batch) / len(batch))
    return sum(avgs) / len(avgs)

def true(data):
    return sum(data) / len(data)
   
def corrected_batch_avg(batches):
    m, n_b, n_m = len(batches), len(batches[0]), len(batches[-1])
    avgs = []
    for batch in batches:
        assert not len(batch) == 0
        avgs.append(sum(batch) / len(batch))
    sum_without_last = 0
    for idx in range(len(avgs)-1):
        sum_without_last += avgs[idx]
    f_1, f_2 = m*n_b/((m-1)*n_b + n_m), m*n_m/((m-1)*n_b + n_m)
    return (sum_without_last * f_1 + avgs[-1] * f_2) / m

print(f'batch_avg: {batch_avg(batches)}')
print(f'true: {true(data)}')
print(f'corrected_batch_avg: {corrected_batch_avg(batches)}')
```
