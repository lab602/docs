# Visual Studio下OpenCV的配置

## 下载Visual Studio

打开[VS官网](https://visualstudio.microsoft.com/)，在Visual Studio IDE下将光标移至Download for Windows，选择Community 2017，即可免费获得最新版的VS社区版。

**在安装VS时，在【工作负载】下必须选择【使用C++的桌面开发】，其他负载根据实际需求下载。此外请确保你拥有微软账号，以便在第一次使用VS时进行登录。** 


![Snipaste_2018-07-21_02-29-50.jpg](https://i.loli.net/2018/07/21/5b522acb64aa7.jpg)

## 获取OpenCV属性表

由于网上配置教程错误较多，这里推荐读者使用我们编写的属性表进行配置。

```props
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ImportGroup Label="PropertySheets" />
  <PropertyGroup Label="UserMacros" />
  <PropertyGroup>
    <ExecutablePath>C:\opencv\build\x64\vc15\bin;$(ExecutablePath)</ExecutablePath>
    <IncludePath>C:\opencv\build\include\opencv2;C:\opencv\build\include\opencv;C:\opencv\build\include;$(IncludePath)</IncludePath>
    <LibraryPath>C:\opencv\build\x64\vc15\lib;$(LibraryPath)</LibraryPath>
  </PropertyGroup>
  <ItemDefinitionGroup>
    <Link>
      <AdditionalDependencies>opencv_world341d.lib;%(AdditionalDependencies)</AdditionalDependencies>
    </Link>
  </ItemDefinitionGroup>
  <ItemGroup />
</Project>
```

移步至[OpenCVProperty](https://github.com/lab602/OpenCVProperty)，点击Download Zip进行下载。

![Snipaste_2018-07-21_02-35-51.jpg](https://i.loli.net/2018/07/21/5b522bbabca82.jpg)

将文件解压到opencv目录中。

![Snipaste_2018-07-21_02-37-29.jpg](https://i.loli.net/2018/07/21/5b522bf6727e6.jpg)

如果你已经拥有git的使用经验，则可以忽略以上的获取方法，只需在Git Bash中输入命令：

```git
git clone https://github.com/lab602/OpenCVProperty.git
```
**获取属性表之后，请务必打开属性表将其中的路径和lib文件名替换为你的设备中实际的路径和lib文件名**

## 配置

打开VS，点击【文件】->【新建】->【项目】，选择Visual C++，创建Windows控制台应用程序。

将Debug调整为x64。

在代码的旁边窗口中选择【属性管理器】

![Snipaste_2018-07-21_02-37-29.jpg](https://i.loli.net/2018/07/21/5b522bf6727e6.jpg)

在项目名称下点击右键

![Snipaste_2018-07-21_02-51-23.jpg](https://i.loli.net/2018/07/21/5b522ff60da25.jpg)

选择【添加现有属性表】，添加“...\opencv\OpenCVProperty\DebugX64”这一路径下的OpenCVProperty.props文件


## 测试

在项目文件夹下添加一张名为“1.png”的图片。在代码窗口中进行编辑：

```c++
#include "stdafx.h"
#include <opencv2/opencv.hpp>

using namespace cv;

int main()
{
	Mat img = iread("1.png");
	imshow("【测试】",img);
	waitKey(6000);
	return 0;
}
```

之后点击上侧的【本地Windows调试器】

![Snipaste_2018-07-21_03-06-04.jpg](https://i.loli.net/2018/07/21/5b5232ba5e9d8.jpg)

若显示图片，则配置成功。



