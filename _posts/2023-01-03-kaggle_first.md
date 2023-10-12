---
layout: post
title: Kaggle第一次比赛心得
date: 2023-01-03
Author: Ursula
tags: [ML]
comments: true
--- 

## 新手避雷
1. 在未组队的情况下私下共享资料属于违规行为，组队截止时间过后尤其不能这样
2. 提交notebook的时候，kaggle的服务器只能找到前两个输出文件，所以一定要把你要提交的文件放在前两个（我们就是犯了这个错误，痛失银牌 cry
3. 防shake，一般来说，ensemble效果差不多而原理不同的模型，既可以提升LB分数，又能防shake，但是加权的weight不要调的过于仔细，否则很可能会过拟合LB


## G2Net Detecting Continuous Gravitational Waves

这场比赛的重点在生成训练数据和噪声处理，模型大家基本都是调用TIMM库，使用efficientNet和Inception，模型和训练方面用到的trick用不多

比赛的收获
- 熟悉kaggle的使用
- 了解比赛规则
- 找到了几个优雅的深度学习的代码模板
  [like this](https://www.kaggle.com/code/batprem/inception-v4-score-boost)


## 分类模型

### Model
Model = encoder+classifier 输出为属于某类的概率，0~1

这次比赛中使用的encoder是GooLeNet，可直接通过TIMM库调用

```python
self.classifier = nn.Sequential(
  nn.Linear(n_features, n_class, bias=True),
  nn.Sigmoid() ## nn.Softmax() 多分类
  )
```

### critrion BCE

nn.BCELoss()

nn.BCEWithLogitsLoss() 自带Sigmoid

这两个函数在计算时都有对p增或减一个较小值防止p=0或1时出现无穷大

## Tricks

### large kernel

[Scaling Up Your Kernels to 31x31: Revisiting Large Kernel Design in CNNs](https://arxiv.org/abs/2203.06717)

> in contrast to small-kernel CNNs, large-kernel CNNs have much larger effective receptive fields and higher shape bias rather than texture bias.

### TTA
flip 翻转

shift 平移

masking 遮住图像的一部分

```python
transforms_time_mask = nn.Sequential(torchaudio.transforms.TimeMasking(time_mask_param=10))

transforms_freq_mask = nn.Sequential(torchaudio.transforms.FrequencyMasking(freq_mask_param=10))

# horizontal flip
img = np.flip(img, axis=1).copy()
# vertical flip
img = np.flip(img, axis=2).copy()
# vertical shift
img = np.roll(img, np.random.randint(low=0, high=img.shape[1]), axis=1)

# tima masking
img = transforms_time_mask(img)
# frequency masking
img = transforms_freq_mask(img)
```

了解更多Augmentation方法可见[博客](https://zhuanlan.zhihu.com/p/41679153)及[论文](https://arxiv.org/abs/1904.08779)

## 好用的库
1. TIMM 计算机视觉模型库
2. optuna 参数调优库
3. wandb 在线可视化库

## Master的黑魔法
1. Random Search，除梯度下降以外的另一种多元最优化方法
2. 发现噪声服从chi分布，尝试生成和测试数据类似的噪声
3. [AE denoising](https://blog.csdn.net/qq_37524214/article/details/107415974) based pretraining

https://www.kaggle.com/competitions/g2net-detecting-continuous-gravitational-waves/discussion/376233
