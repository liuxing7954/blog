---
title: Spring-IOC(1)-意义

---
## 什么是IOC

Spring Core 的`核心部分`

> 先行知识 -- 依赖注入`Dependency Inversion`

#### 使用情况

一个`上级依赖于下级`的架构

![1556338180523](C:\Users\liuxi\AppData\Roaming\Typora\typora-user-images\1556338180523.png)

如果最下级的Tire的成员变量size要改为`动态化`的，那架构图如下

![1556338298535](D:\System\Doc\CODE\blog\source\_posts\1556338298535.png)

问题不言而喻，所以优化后的架构应该是`下层依赖于上层`，即将下层的对象`注入`到上层中，将程序进行`解耦`。

优化后的架构为

![1556338486389](D:\System\Doc\CODE\blog\source\_posts\1556338486389.png)

此时再进行相同的需求改造时,只要改变Tire类的代码.

![1556338534655](D:\System\Doc\CODE\blog\source\_posts\1556338534655.png)

两者的差别很明显,`耦合度后者更低`,但是后者的初始化代码变的非常复杂.

为了解决这个问题,IOC容器内部隐藏了这些步骤.即将Luggage的初始化简化成`前者情况`的样子.

IOC容器内部初始化一个对象的过程为

![1556338766562](D:\System\Doc\CODE\blog\source\_posts\1556338766562.png)

