# Python环境下OpenCV的配置

## Python的安装与配置

Python的安装与配置本手册不再赘述，请参考廖雪峰的Python3教程[安装Python](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014316090478912dab2a3a9e8f4ed49d28854b292f85bb000)一节。

**需要注意的是，在安装Python时，请不要安装最新的3.7.0版本，这有可能导致pip安装opencv失败**

## OpenCV安装

在Windows自带的PowerShell或cmd中输入命令：

```pip
pip install opencv-python
```

由于众所周知的原因，Python官方的pip源在下载时速度比较慢。若实际下载速度较慢，我们可以进行换源。推荐[清华大学TUNA](https://mirrors.tuna.tsinghua.edu.cn/)源。换源只需输入命令：

```pip
pip install opencv-python -i https://pypi.tuna.tsinghua.edu.cn/simple
```
在python环境下，我们输入:

```python
import cv2
```
![Snipaste_2018-07-21_03-30-38.jpg](https://i.loli.net/2018/07/21/5b52386fe48cc.jpg)

若无异常提示，则安装成功。
