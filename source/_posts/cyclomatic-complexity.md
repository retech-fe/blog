---
title: 圈复杂度Cyclomatic complexity
date: 2023-03-06 15:13:02
categories: 工程化
tags: eslint
excerpt: 在这篇文章中，讲了圈复杂度的概念、计算公式及如何配置eslint开启圈复杂度，如何解决圈复杂度过高的问题。
author: pengfei.zuo
---


## 圈复杂度是什么

+ 圈复杂度是公认的衡量代码质量的重要标准，记作V(G)
+ 它主要反映代码中的路径数量，它的值越大，表示代码越难测试和维护

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/06/14-54-22-78548692f412e5904d0cdb650cb3f141-20230306145422-04147e.png)

## 如何计算圈复杂度

通常使用两种方式来计算V(G)

***方式一***

V(G) = e - n + 2

其中，`e`表示流程图中的边数，`n`表示流程图中的节点数

***方式二***

V(G) = R

其中，R表示平面被流程图划分的区域数量


示例一

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/06/14-58-30-35e2be49e73290a1028b76d1e847330b-20230306145830-645e85.png)

示例二

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/06/14-59-51-09da9f88755c191baecab8174da3c497-20230306145951-82f7b0.png)

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/06/15-01-09-7a5cc01e5c878b8cd28ac1b434e1dee1-20230306150109-2ea885.png)

示例三

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/06/15-02-10-eb43a94fcc8b53e360a080a72698c1ed-20230306150210-eaeedc.png)

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/06/15-02-56-9e58887695dbef50aa4555278893f32d-20230306150256-1d8dd9.png)

## 如何配置eslint圈复杂度

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/06/15-05-04-36e0b4502e2b6ae789d6b90dcbd10ad4-20230306150503-2ec51a.png)

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2023/03/06/15-05-47-a067e2a5164d64bf11d94e0c90a63a1c-20230306150547-66cf07.png)

## 解决方案

+ 对象映射
+ 提取函数，降低每个函数的圈复杂度