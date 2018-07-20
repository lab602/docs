# OpenCV的下载与安装

## 下载和安装OpenCV SDK

这里以Windows 10操作系统为例。

在[OpenCV官网Release](https://opencv.org/releases.html)中点击最新版本的Win Pack进行下载。

![01.jpg](https://i.loli.net/2018/07/21/5b52259975875.jpg)

按照完成OpenCV后，几个GB的文件就解压到了我们指定的目录下。在这里进行简单的说明：解压完成后会在指定的路径下生成一个名为OpenCV的文件夹，它包含了两个子文件夹，分别为build和sources。其中，build文件夹中是支持OpenCV使用的相关文件，而sources中为OpenCV的源代码以及相关文件。OpenCV官方示例以及说明文档可以在sources文件夹下的samples里找到。

## 配置环境变量

配置方法如下：

点击【计算机】->【（右键）属性】->【高级系统设置】->【高级（标签）】->【环境变量】->【（选中系统变量中的）PATH】->【（点击）编辑】->【（点击）新建】

然后添加"...opencv\build\x64\vc15\bin"即可。

![Snipaste_2018-07-21_02-23-12.jpg](https://i.loli.net/2018/07/21/5b5228a7e7895.jpg)