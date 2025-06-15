---
author: KDQ
date: 2025/06/13
email: kdq1290@gmail.com
tags:
  - SLAM
  - fastlio2
  - eskf
---

# 背景

FAST-LIO2使用的是迭代式误差kalman 滤波算法，这里的状态量都是误差状态，用误差状态进行状态估计的好处有三：
	1） 误差状态可以用最小自由参数化，避免过参数化（用李代数的加减来替代李群的变化）
