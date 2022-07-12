# 图床

在Markdown编写过程中，不可避免的需要插入图片。

有一种极其繁琐的操作是：先是把本地图片放到项目source/img目录下，然后push到github触发自动部署，等成功后复制图片地址，最后再回到Markdown修改图片链接。极度不舒适！

如果你有阿里云、腾讯云等厂商的oss；AWS的S3；那更好，如果没有可以用下面的方案：


## PicGo

### 介绍
[PicGo](https://molunerfinn.com/PicGo/)一个用于快速上传图片并获取图片 URL 链接的工具。
支持剪贴板上传，只有截图后复制到剪贴板就可以支持上传。![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/07/12/13-10-45-11d3a4d82ef94465b4d1e90af633c3fb-20220712131044-ca291d.png)

### 下载
[下载地址](https://github.com/Molunerfinn/PicGo/releases);可以下载稳定的最新版，![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/07/12/13-07-00-5028f1118b2eb733ad72036514baf438-20220712130700-5886b6.png)

### 配置

具体配置可以查看[官方文档](https://picgo.github.io/PicGo-Doc/zh/guide/config.html#github%E5%9B%BE%E5%BA%8A)。

首先切换到![github图床](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/07/12/14-23-09-77bdb2b495c15542ac342c6481afba8a-20220712142308-9f0a2b.png)；可以把其他图床关掉，只留下github

+ token需要配置你的token；[如何生成token](https://picgo.github.io/PicGo-Doc/zh/guide/config.html#github%E5%9B%BE%E5%BA%8A)，上面的文档第三步
+ 仓库名: `retech-fe/image-hosting`
+ 分支名: `main`
+ 存储路径: `img/`

![](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/07/12/13-14-30-9ce2dd5f6af0e658cf66a862e3cd4942-20220712131430-6511eb.png)

如果觉得上传后得文件名比较长可以下载安装插件[picgo-plugin-rename-file](https://github.com/liuwave/picgo-plugin-rename-file#readme)。

文件名格式可以配置如下：
```
{y}/{m}/{d}/{h}-{i}-{s}-{hash}-{origin}-{rand:6}
```

### 如何使用图片

上传完图片后，可以切换到![相册](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/07/12/14-29-38-d0076907599c1e003a0ca7a18ada3f88-20220712142937-6ae996.png)，复制链接

在Markdown的用法如下：

```
![相册](https://raw.githubusercontent.com/retech-fe/image-hosting/main/img/2022/07/12/14-29-38-d0076907599c1e003a0ca7a18ada3f88-20220712142937-6ae996.png)
```


## 其他方案

### Boomb

[Boomb](https://boomb.cn/login)


### picx

[picx](https://picx.xpoet.cn/#/upload)

