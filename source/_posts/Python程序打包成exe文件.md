---
title: Python程序打包成exe文件
route: python-packexe
date: 2020-02-03 10:45:31
tags: Python
categories: Python
image: /images/cover/python-packexe.png
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Python作为现在一门很火的编程语言，无论是不是计算机专业的都有爱好者在学习。我们一般写的Python程序，都是只能在我们自己电脑上进行运行，有时候想分享给小伙伴用一下，但是不是每个小伙伴电脑上都安装有运行Pythonde环境。这个时候我们可以将我们的写好的Python程序打包成exe文件发送给小伙伴，无论对方电脑是有没有Python环境，都可以直接双击运行。下面以Windows环境记录一下打包过程。

<!-- more -->

# Python程序打包成exe文件

## 1. 准备工作

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**PyInstaller** 是一个十分有用的第三方库，可以用来打包 python 应用程序，打包完的程序就可以在没有安装 Python 解释器的机器上运行了。它能够在 Windows、Linux、 Mac OS X 等操作系统下将 Python 源文件打包，通过对源文件打包， Python 程序可以在没有安装 Python 的环境中运行，也可以作为一个 独立文件方便传递和管理。PyInstaller 支持 Python 2.7 / 3.4-3.7。可以在 Windows、Mac OS X 和 Linux 上使用，但是并不是跨平台的，而是说你要是希望打包成 .exe 文件，需要在 Windows 系统上运行 PyInstaller 进行打包工作。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;安装PyInstaller

```bash
pip install pyintaller
# 或者
python -m pip install pyinstaller 
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;安装成功如下。

![安装成功](安装pyinstaller.png)

## 2. 打包EXE

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先进入cmd，进入到你的python文件的目录下,执行命令如下。

```bash
pyinstaller [参数] xxx.py
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参数说明

+ **-i：**添加一个应用图标，ico文件。(只对Windows系统有效)。
+ **-F：**打包成单独的`.exe`文件。
+ **-W：**无控制台运行界面。
+ **-D：**创建一个目录，里面包含exe以及其他一下依赖性文件
+ 注：**pyinstaller -h**：查看参数。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我的是hello.py程序，执行下面操作。

```bash
pyinstaller -F hello.py
```

![打包exe](打包exe.png)

**PyInstaller**会对脚本进行解析，解析过程如下：

1、在脚本目录生成hello.spec文件；

2、创建一个build目录；

3、写入一个日志文件和中间流程文件到build目录；

4、创建dist目录；

5、生成可执行文件到dist目录。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;执行完成之后就可以在dist目录下找到我们打包好的`exe`文件，双击就可以直接运行了。

![运行](运行.png)

## 3. 引入外部文件

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;按照以上步骤打包exe的话，当你的代码中需要调用一些图片和资源文件的，这是不会自动导入的，需要你自己手动复制进去才行。不然 exe 文件运行时命令窗口会报错找不到这个文件。

导入方法：假设`hello.py`程序中需要引入一个`test.txt`文件，`test.txt`文件和`hello.py`文件位于同一路径下。首先我们运行：

```bash
pyi-makespec -F hello.py
```

此时会生成一个`.spec`，这个文件会告诉pyinstaller如何处理你的脚本，pyinstaller创建一个exe的文件就是依靠它里面的内容进行执行的。正常情况下是不需要修改这个`.spec`文件的，除非你需要打包一个 dll 或者 so 文件或者其他数据文件。

接下来我们就修改这个`.spec`文件：

```spec
a = Analysis(['hello.py'],
             pathex=['G:\\CodeSpace'],
             binaries=[],
             datas=[],   ### <------ 修改
```

修改为：

```spec
a = Analysis(['hello.py'],
             pathex=['G:\\CodeSpace'],
             binaries=[],
             datas=[('test.txt','.'),],  ### <-----修改此处添加外部文件
```

然后再生成`exe`文件：

```bash
pyinstaller hello.spec
```

这样生成的`exe`文件就可以正常引入外部文件了。

## 4. 注意事项

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1、直接运行最终的 .exe 程序，可能会出现**一闪而过**的情况，这种情况下要么是程序运行结束（比如直接打印的 helloWorld），要么程序出现错误退出了。这种情况下，建议在命令行 cmd 下运行 .exe 文件，这时就会有文本输出到窗口。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2、`-i` 是改变图标的，图标在不同情况下(比如资源管理器文件列表前面的图标、桌面、开始菜单等)需要不一样尺寸的图标。如果尺寸不合适的话，可能出现有的地方显示正确有的显示不正确的情况。Windows的图标大小尺寸有128\*128、64\*64、48\*48、32\*32、16\*16，请正确选用。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3、写代码的时候应当有个良好的习惯，用什么函数导什么函数，不要上来 import 整个库，不然最后你会发现你一个 100KB 的代码打包出来有 500MB；

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4、在引入外部文件时，如果需要引入多个文件，`.spec`文件修改如下：

```spec
a = Analysis(['hello.py'],
             pathex=['G:\\CodeSpace'],
             binaries=[],
             datas=[('test_1.txt','.'),('test_2.txt','.'),('XXX/test_3.txt,'.'),],
```

## 5、总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上就是使用PyInstaller打包流程进行简单的介绍和使用，更多内容可以参见官方文档：[https://pyinstaller.readthedocs.io](https://pyinstaller.readthedocs.io)。