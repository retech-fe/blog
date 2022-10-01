---
title: Docker中ADD和COPY的区别
date: 2022-09-10 12:05:02
categories: DevOps
tags: CICD
excerpt: 在这篇文章中，对`Docker`中`ADD`和`COPY`的这两个命令进行了解释和对比，你将知道如何选择来使用它们。
author: pengfei.zuo
---

## 背景
当我们在使用`Dockerfile`创建镜像时，你可以使用两个命令来进行文件或者目录的复制：`ADD`和`COPY`。它们的用途基本上是一致的除了有一些小的不同点。

于是，为什么会有两个命令呢？我们在什么情况下使用它们呢？

在这篇文章中，会对每个命令进行解释以及分析`Docker`中`ADD`和`COPY`的对比，你将知道如何选择来使用它们。

## Docker ADD 命令

`ADD` 指令在`Docker`平台出现得比较早，在早期就已经是命令集合中的一员了。`ADD`主要是复制文件和目录到特定容器下的文件系统中。

`ADD`命令的基本语法如下：

```
ADD <src> … <dest>
```

### 复制文件

它包含你想要复制的源文件（`src`）以及你需要存储的目的地（`dest`）。如果给定的`src`是一个目录的话，`ADD`命令将会复制目录下的所有文件。

例如，如果文件存在于本地，并且你想要复制到一个目录，你可以这样：


```
ADD /source/file/path  /destination/path
```

### 远程复制文件

`ADD`命令同时也支持从一个远程地址复制文件。它将会从远程下载文件，并且复制到容器内的文件系统，例如：

```
ADD http://source.file/url  /destination/path
```

### 本地解压文件并复制文件

如下，指定本地的压缩文件，以及要复制的容器目标文件系统路径即可：

```
ADD source.file.tar.gz /temp
```

这里需要注意的是，你不能从远程地址下载和解压缩文件或者目录。

##  Docker COPY命令

基于有些功能问题，`Docker`不得不增加另外的命令来复制内容，也就是`COPY`命令。

不同于`ADD`命令，`COPY`指令仅仅只有一个功能。它的角色就是复制本地的文件或者目录到容器的文件系统。也就意味着它不会去解压缩一个压缩的文件，仅仅只是复制而已。

这个指令只能用在处理本地的文件。也就是说，你不能用它来复制远程路径的文件到容器内。

下面是使用 `COPY` 命令的基本语法：

```
COPY <src> … <dest>
```

For example:

```
COPY /source/file/path  /destination/path
```

## Docker Copy vs ADD

那为什么`Docker`需要添加一个新的，并且相近的命令？

事实就是 `ADD` 命令在实践过程中有很多的功能问题出现，并且它极其的不可预测。这个不稳定的表现结果经常是出现在你复制内容的时候，到底是需要压缩的内容还是解压缩内容。

`Docker`由于有很多存在的用途，并不能完全替代 `ADD` 命令。同时，为了向后兼容，最好的稳妥做法就是添加 `COPY` 命令。


### 到底如何选择？

Docker的官方文件有提到，应当尽可能的使用 `COPY`，原因是相对于 `ADD`，它的使用更加透明。

如果你需要复制本地的内容到容器内，最好用 `COPY`。

`Docker`团队同时也强烈建议不要使用 `ADD` 来下载和复制远程地址文件，而是使用更加安全和高效地通过 `RUN` 命令来使用 `wget` 或者 `curl` 达到同样的功能。如果这样操作的话，你将避免创建新的镜像层，以及减少镜像大小。

例如，你想要下载远程地址的一个压缩文件，解压缩内容，并在最后清除下载包。

使用 `ADD` 的方式：

```
ADD http://source.file/package.file.tar.gz /temp
RUN tar -xjf /temp/package.file.tar.gz \
  && make -C /tmp/package.file \
  && rm /tmp/ package.file.tar.gz
```

你应该这样做：

```
RUN curl http://source.file/package.file.tar.gz \
  | tar -xjC /tmp/ package.file.tar.gz \
  && make -C /tmp/ package.file.tar.gz
```

##  总结

总结来说，尽量使用 `COPY` 命令。就像`Docker`官方的建议一样，尽量避免使用 `ADD` 命令，除非你需要复制一个本地压缩文件并在容器内进行解压缩。
