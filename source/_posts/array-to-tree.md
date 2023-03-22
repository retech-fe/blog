---
title: 扁平数据结构转Tree
date: 2023-03-22 15:55:02
categories: 算法
tags: 
excerpt: 在这篇文章中，讲了扁平数据结构转Tree的两种方法
author: haida.wang
---


## 题目

打平的数据内容如下：

```JavaScript
let arr = [
    { id: 1, name: '部门1', pid: 0 },
    { id: 2, name: '部门2', pid: 1 },
    { id: 3, name: '部门3', pid: 1 },
    { id: 4, name: '部门4', pid: 3 },
    { id: 5, name: '部门5', pid: 4 },
];
```

输出结果：

```JavaScript
[
    {
        "id": 1,
        "name": "部门1",
        "pid": 0,
        "children": [
            {
                "id": 2,
                "name": "部门2",
                "pid": 1,
                "children": []
            },
            {
                "id": 3,
                "name": "部门3",
                "pid": 1,
                "children": [
                    {
                        "id": 4,
                        "name": "部门4",
                        "pid": 3,
                        "children": [
                            {
                                "id": 5,
                                "name": "部门5",
                                "pid": 43,
                                "children": []
                            }
                        ]
                    }
                ]
            }
        ]
    }
]
```

## 方法 1

```JavaScript
function arrayToTree(arr) {
    // 循环所有元素
    arr.forEach((a) => {
        // 给当前元素 a 添加 children 属性
        a.children = [];

        // 循环所有元素
        arr.forEach((b) => {
            // 判断当前元素 b.pid 是否等于 a.id，如果相等，则为 a 的子级
            if (b.pid === a.id) {
                a.children.push(b)
            }
        })
    })

    // 查找所有 pid 等于 0 的元素
    return arr.filter((a) => a.pid === 0);
}
```

通过两次循环，利用引用类型的特性给所有数组添加自己的子级，再查找 pid 等于 0 的所有元素


## 方法 2

```JavaScript
function arrayToTree(arr) {
    // 存放结果
    const result = [];
    // 以 id/pid 为 key 存放元素
    const itemMap = {};

    // 循环所有元素
    for (const item of arr) {
        // 取 item 中的 id 和 pid
        const { id, pid } = item;

        // 先以 id 为 key 存入 Map 中
        if (!itemMap[id]) {
            itemMap[id] = {
                children: []
            }
        }
        itemMap[id] = {
            ...item,
            children: itemMap[id].children,
        }

        // 再从 Map 中取出元素
        const treeItem = itemMap[id];

        // 如果 pid 为 0，则直接放到结果中
        if (pid === 0) {
            result.push(treeItem);
        }
        // 否则以 pid 为 key 存储 Map 中
        else {
            if (!itemMap[pid]) {
                itemMap[pid] = {
                    children: [],
                }
            }
            // 将元素放入以 pid 为 key 的元素中的子级
            itemMap[pid].children.push(treeItem);
        }
    }

    return result;
}
```

通过 Map 类型，以 id/pid 为 key，一次循环即可将所有元素的父子关系添加完成