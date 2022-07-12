# 贡献指南

## fork仓库

+ [原始仓库](https://github.com/retech-fe/blog)
+ 目标仓库：fork 到自己的`github`上
  

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/07/12/14-52-44-261cedfc761b810dd109873686284081-20220712145244-19cac2.png)


## 拉取仓库

fork仓库后会在自己的账号下看到blog的仓库；然后clone代码，切换到main分支, 

## 运行项目
  1. 切换分支去main `git checkout main`
  2. 通过yarn或者npm命令进行安装依赖包
  3. 通过命令 `yarn server`启动项目

**注意**

 > 每次启动项目前尽量先清除缓存`yarn clean `
 > 如需切换端口号： `yarn server -p 5000`

## 编写文章

1. 运行`hexo new 'xxx'`，会默认在source/_posts目录下面生成一个xxx.md 模板
2. 打开xxx.md文档，进行文章配置

### 初始化配置

 **默认生成：**

```yaml
---
title: title
date: {{ date }}
#分类
categories:
#标签
tags: 
excerpt:
---
```

示例：

 ```yaml
 ---
 title: 搭配 Fluid 如何优雅的写一篇文章
 author: Vince
 date: 2021-07-11 17:39:30
 category: 主题示例
 tags: 
   - 用户经验
   - 示例
   - Fluid
 excerpt: Fluid 是一款很优雅的主题，主要介绍从使用主题拓展和配图来写作。
 ---
 ```

#### category可选规范

- 基础技术，前端领域的基础技术内容，如 ECMAScript、TypeScript、WebAssembly、Node 等等

- 前端框架，前端流行框架如 React、Vue、Angular 等的最新动态、技术分析、生态分享等等
- 桌面开发，PC 桌面跨平台软件开发技术，如 NW、Electron、tauri 等等
- 工程化，工程化相关技术动态，如 Webpack、Vite、rollup、ESbuild、Parcel 等等；构建部署等
- 跨端框架技术，跨端与跨框架的技术动态，如Flutter、RN、小程序跨端解决方案动态等等
- 低代码，低代码无代码的技术动态，如配置化、动态化、建站、移动应用低代码开发等等
- 服务端开发，与前端紧密相关的服务端技术动态，如传统服务端开发、Severless 等等
- 图形编程，图表、动画、游戏、AR/VR 等技术动态
- 人工智能，AI 相关技术动态，如深度学习等等
- 团队管理，技术管理相关方法论
- 设计哲学，关于工作、开发、设计的思考

- 工具推荐，介绍业界有意思、好玩的工具

#### tags可选规范

**例如：** ECMAScript、TypeScript、WebAssembly、Node， React、Vue、Angular，Webpack，Flutter，微前端等等。

### 开始编写

在基础配置信息下面编写正文，支持html和mackdown

- [ 搭配 Fluid 如何优雅的写一篇文章](https://retech-fe.github.io/blog/2021/07/11/fluid-write/)

> 编写完成后push 到自己的远程main分支



## 发起Pull request

1. 

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/qYhT4KYY4b.jpg)

2. 

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/20220712170101.png)

3. 点击 `Create pull request`

4. 通知仓库owener 进行 Review，若不通过，请按照owener提供的意见进行修复，再次push
