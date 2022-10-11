---
title: 一周学习记录 2
date: 2022-10-11 15:10:39
tags: 
- MaskFormer
- Mask Classification

categories: Weekly Report
---

# Maskformer

这周读了maskFormer的论文，Experiment部分没看准备留到看code的时候一起。

## Idea 

作者们在这篇文章中的想法很简单：*mask classification is sufficiently general to solve both semantic- and instance level segmentation tasks*.   
这里我们区分两个概念：per-pixel classification 和mask classification， 在文章的intro里我们可以得知，在先前的Dense classification任务中通常是沿着这两种思路来进行，在最早时候，mask classification的方法占有重要地位，诞生了诸如O2P, SDS之类的SoTA模型。然而在FCN诞生后，人们的关注点逐步转移至per-pixel classification的思路上，此后很多模型都是基于FCN的形式构建的，诸如MaskRcnn。然而per-pixel classification based的模型有缺陷，主要是：

- 分类数目固定
- 开销大（相对于mask classification，个人猜测） 

然而以往的mask classification的方法往往需要各种限定条件，诸如需要生成bounding box、或者auxiliary loss，而且分割效果不一定有per-pixel的方法好。
在maskFormer架构里，作者提出的架构则实现了一种较为高效的mask classification方法，作者认为Mask classification 的方法已经足够完成segmentation任务。

## Architecture
Maskformer的思路是将per-pix classification based的方法改造成mask classification based的方法。这里是架构图
<img src = 'architecture.png' align='center'>

就说这个架构主要还是三个模块：
- per-pixel module
- transformer module
- segmentation module

### per-pixel module

获取feature map，生成mask，就是说这个backbone可以是任何特征提取器，transformer based或者是CNN都行，就是先前待改造的per-pixel方法。这里的Decoder Upsample后并没有直接生成mask，而是生成了一个per-pixel embedding(这里的embedding有点不解是具体指什么，是不是可以理解为特征图在另一个embedding space保留了原信息的形式？那这样上采样的意义是什么？仅仅就是变换以下dimension？似乎是这样，要还原到原来的大小，不管这个embedding的意义好多，需要请教一下)。

### transformer module

接受feature map 和 对应 N 个categories 的 Query，建立各个mask之间的联系。输出 per-segment embedding Q，编码了全局信息。

### segmentation module

生成mask和classification，用一个两层的MLP，输入Q，得到N个segments的categories， 并且生成N个mask的mask embedding，用这N个embedding和先前的per-pixel embedding求积获得mask。训练时这两部分各有两个loss: $\mathcal L_{classification}$ 、$\mathcal L_{mask}$,前者交叉熵，后者用一般的mask loss，总得loss为: $ \mathcal L = \mathcal L_{classification} + \mathcal L_{mask}$

最后的到的分类结果和segments输入到相应的头里，这里的头更多取决于Metirc而不是任务，文中提了两种头:
#### General Head

通用头,有时间写一下。

#### segment Head

专门为semantic segmentation设计的头，有矩阵加速

## 查缺补漏

mask loss函数

## last

**觉得厉害的的:** 
- 改造的想法可以的，现有模型加以扩充还是会有意想不到的效果  

**有点不理解的：**
- transformer中的decoder的Query有什么作用，文章说是为每一个类别N生成一个Q，然后输出就相当于建立了各个segment之间的global info，可是这个Query怎么来的？难道像Vit里面的class token也是可学习的？
- 生成mask的过程为什么不直接由Decoder生成，难道是通过结合全局信息Q可以帮助改善mask生成效果？
- 什么时候才能摆脱FCN这种Mask生成的方法哇😭，难道这个是最好的？




