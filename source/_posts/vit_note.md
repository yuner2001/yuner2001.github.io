---
title: 一周学习记录 1
date: 2022-09-27 15:51:50
tags: 
- transformer
- attention mechanism

categories: Weekly Report
---
&nbsp;&nbsp;&nbsp;&nbsp;推免的事终于尘埃落定啦，奈何还有大软要写文档，当然no matter，该干的事情还是得干滴。师姐这周推荐了一篇文章，讲maskformer。第一次进行一些这样的小小研究还有些不适应呢，所以主要补习了下前置知识。这周主要看的还是Transformer和注意力机制相关的东西，在这里记录一下。

# 了解的资讯
## Attention mechanism

目前接触的特殊注意力机制有主要有几种：

- self-attention
- multi-attention

一般的注意力机制就是说要具备三个东西，分别是 $K$ (key) 、$Q$ (query) 、$V$ (value),其中 一个 $K$ 可以映射到一个 $V$，组成一个k-v对。整个计算过程就是输入一个查询序列q,用一个衡量相似度的东西($\alpha$)计算以下这个q和各个key的相似程度，然后把这个权重softmax归一化后与对于的v相乘相加（就是个加权和）。

简洁来说就是：
1. 输入 通过embbeding layer 后记作，$I \in R(N,d) $，分别由三个transforming matrix $W^k$ $W^q$ $W^v$ 变换到矩阵 $K$ $Q$ $V$。（构建kqv）
2. $A = K^TQ$。(求权重参数) 
3. $\hat{A} = softmax(A)$。（归一化以下显得像概率一些，稳定点，省的捅出些梯度爆炸啥的幺蛾子）
4.  $ B = \hat{A}V $。(算出输出，这里要不要转置还要看具体情况，周记这种写给自己看的东西表意就行啦)

然后呢就是说，self-attention就是输入三个东西都是同一个序列自己， multi-attention就是说在上面过程的第一步后还要在乘上另外的transforming matrix来弄出不同的权重来（头），要几个头就整几个矩阵。                                                                                                                                     
### transformer
架构看原文，那个图画的可以的。主要要注意的是：
- mask multi-attention这个玩意儿，在进行attention的过程里，在用kqv矩阵去算权重后要将未softmax的 $\hat{A}$ 给他乘上一个mask矩阵（一般是上三角，当然只要使得两者乘出来的结果矩阵上三角是一个合适的负数即可）。
- position embedding，用来添加位置信息来达到rnn效果，否则和CNN没啥区别，具体计算是可以人工或者学习，人工的话自己规定大小然后用那个三角公式去算（三角好处是可以度量相对距离，数值在[-1,1]稳定）
- 这里注意力分数采用scale-dot
- FNN网络是运用在输出序列的输入序列的各个样本上（就是说一个vector一个vector的输入，原文是先投影到高维然后又降维到原来的dimension）、
- 加入残差连接防止模型退化（至少扩大模型规模和性能成严格正比），normalization用的不是BN而是LN

## 记点有用的
最后总结一下学习的感想：
- 为啥用多头：模拟cnn多通道的效果，提取不同层次

- BN、LN： BN是从feature出发，在序列长度不一的情况下方差期望不稳定容易震荡；LN是从样本出发，适合处理文本序列这种输入样本长度可能不一的任务。

- 线性层可以投影，但是无法拟合变量之间的关系(这个和上面无关，是看💐搞得顺便记下)

- 信息从数据点到另一个点距离短，信息损失越少，可以PE这种粗暴的相加也传递信息，设计网络时要考虑各方面的feature，最好说设计好feature的时候问问自己从这些feature里自己是不是可以得出差不多的结果