# 内存

## 内存屏障（Memory barrier）

为什么会有内存屏障？
每个CPU都会有自己的缓存（有的甚至L1,L2,L3），缓存的目的就是为了提高性能，避免每次都要向内存取。但是这样的弊端也很明显：不能实时的和内存发生信息交换，分在不同CPU执行的不同线程对同一个变量的缓存值不同。

c++用volatile关键字修饰变量可以解决上述问题。

## 参考

1. <https://zhuanlan.zhihu.com/p/43526907> volatile与内存屏障总结
2. <https://www.dpdk.org/blog/2019/08/21/memory-in-dpdk-part-1-general-concepts/> DPDK 中的内存
