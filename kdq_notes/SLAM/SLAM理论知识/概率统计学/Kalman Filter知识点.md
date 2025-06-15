
### 1. 关于Kalman Filter的几个问题
- 过程噪声都包含哪些噪声
	过程噪声包括建模误差，跟你计算机[舍入误差](https://www.zhihu.com/search?q=%E8%88%8D%E5%85%A5%E8%AF%AF%E5%B7%AE&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A153405833%7D)（比如说用了[浮点](https://www.zhihu.com/search?q=%E6%B5%AE%E7%82%B9&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A153405833%7D)，那么会引入1e-7的误差项）、模型的线性化程度、[离散化](https://www.zhihu.com/search?q=%E7%A6%BB%E6%95%A3%E5%8C%96&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A153405833%7D)引入误差有关系；
