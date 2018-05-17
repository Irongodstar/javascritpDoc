# Dva 数据获取的流程分析





## 原理架构图

![](./images/data.png)


根据官方提供的dva原理的基础, 并结合我们现在使用的构架进行重新绘制.


## 数据获取流程

我们构架是如何获取数据的呢? 我们下面会以获取用户列表为例子, 我们将走一边这个流程.





1. router
2. model
    1. subscription
    2. call
    3. axios
    5. effect
    6. reducer
3. connect component
