---
title: Test
date: 2025-10-28
categories: 
- test
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## AHP

我们先构建层次结构：

- **目标层（Goal）**：想实现的最终目标。	
- **准则层（Criteria）**：影响目标的因素，可以是多个指标。
- **方案层（Alternatives）**：最终要选择的备选方案。

本题中：

- **目标层**：排序所有项目（存疑）。
- **准则层**：（这里写各项指标）。
- **方案层**：（这里写各个项目）。

然后构建判断矩阵 $A$（指标个数 $\times$ 指标个数，即 $n\times n$），使用 1-9 标度法：

| 数值           | 含义                                                 |
| -------------- | ---------------------------------------------------- |
| $1$            | 两者同等重要                                         |
| $3$            | 轻微更重要                                           |
| $5$            | 明显更重要                                           |
| $7$            | 强烈更重要                                           |
| $9$            | 极端更重要                                           |
| $2,4,6,8$      | 上述判断相邻值的两者之间                             |
| $[1,9]$ 的倒数 | 若两元素中后者重要性大于前者，则交换顺序打分并取倒数 |
|                |                                                      |

通过几何平均得到 $w'$
$$
w'_i=(\prod_{j=0}^{n-1}a_{i,j})^{\frac{1}{n}}
$$
将 $w'$ 归一化得到权重向量 $w$
$$
w_i=\frac{w'_i}{\sum_{j=0}^{n-1}w'_j}
$$
因为判断矩阵可能存在主观偏差，所以我们还需要检验权重向量的逻辑一致性。先求最大特征根 $\lambda_{\max}$，其满足
$$
A\cdot w=\lambda_{\max}\cdot w
$$
所以
$$
\lambda=\frac{(A\cdot w)_i}{w_i}
$$
最后得到 $\lambda_{\max}$
$$
\lambda_{\max}=\frac{\sum_{i=0}^{n-1}\lambda_i}{n}
$$
现在来计算一致性指标 $\mathrm{CI}$
$$
\mathrm{CI}=\frac{\lambda_{\max}-n}{n-1}
$$
查表得到随机一致性指标 $\mathrm{RI}$，然后计算一致性比率 $\mathrm{CR}$
$$
\mathrm{CR}=\frac{\mathrm{CI}}{\mathrm{RI}}
$$
若 $\mathrm{CR} <0.1$，则矩阵一致性可以接受，否则需要调整判断矩阵。

最终，我们得到了 AHP 算出的权重向量 $W_{1}=w$，之后拿来跟 熵权 + TOPSIS 算出的权重向量 $W_2$ 做组合赋权。
