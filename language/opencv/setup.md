# OpenCV Setup

- [OpenCV Setup](#opencv-setup)
  - [Use OpenCV on WSL](#use-opencv-on-wsl)
    - [Install OpenCV on WSL](#install-opencv-on-wsl)
    - [Install VcXsrv](#install-vcxsrv)
  - [Use OpenCV on Windows](#use-opencv-on-windows)
    - [Install OpenCV on Windows](#install-opencv-on-windows)
    - [Add to Path](#add-to-path)
    - [Use OpenCV in Visual Studio](#use-opencv-in-visual-studio)
    - [Install OpenCV using vckpg in Visual Studio](#install-opencv-using-vckpg-in-visual-studio)

## Use OpenCV on WSL

> WSL + OpenCV + VcXsrv + VSCode
> 关于WSL以及终端的设置可以参考[这里](https://github.com/Zhytou/CS-Notes/blob/main/tool/terminal/setup.md)

### Install OpenCV on WSL

``` bash
# Install minimal prerequisites (Ubuntu 18.04 as reference) 
sudo apt update && sudo apt install -y cmake g++ wget unzip libgtk2.0-dev pkg-config
# Download and unpack sources
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.x.zip
unzip opencv.zip
# Create build directory
mkdir -p build && cd build
# Configure
cmake  ../opencv-4.x
# Build
cmake --build .
# Install
make install
```

### Install VcXsrv

> VcXsrv是一个开源软件，它可以为wsl的程序提供GUI窗口。

- [下载](https://sourceforge.net/projects/vcxsrv/)安装包

- 配置WSL环境

``` bash
# Edit config for console
vim ~/.bashrc or vim ~/.zshrc
# Add following to the end of file
export DISPLAY=:0
# Save and put the config into work
source ~/.bashrc or source ~/.zshrc
```

- 启动VcXsrv: 在安装目录，双击xlaunch启动服务。此时，在WSL编译运行一个OpenCV程序就可以看到图形界面了。

## Use OpenCV on Windows

> OpenCV + Visual Stdio
> 关于VS的更多使用介绍可以看[这里](https://github.com/Zhytou/CS-Notes/blob/main/tool/vs/vs.md)

### Install OpenCV on Windows

- 前往[官网](https://opencv.org/releases/)下载安装包；

- 双击运行安装。

### Add to Path

- 打开高级系统设置-高级-环境变量，将`D:\opencv\build\x64\vc16\bin`添加至环境变量中。此处包含运行OpenCV所需的动态链接库。

### Use OpenCV in Visual Studio

- 在项目-属性-VC++包含目录-包含目录中添加OpenCV头文件路径；

- 在项目-属性-VC++包含目录-库目录中添加OpenCV库路径；

- 在项目-属性-链接器-输入-附加依赖项中添加OpenCV库名称。

### Install OpenCV using vckpg in Visual Studio

> 关于vckpg包管理器的更多使用介绍可以看[这里](https://github.com/Zhytou/CS-Notes/blob/main/tool/vckpg/basis.md)

使用vckpg相比直接使用OpenCV官网安装器管理和配置都更方便，具体操作如下：

``` powershell
# 安装vckpg
git clone https://github.com/microsoft/vcpkg
.\vcpkg\bootstrap-vcpkg.bat

# 使用安装opencv
.\vcpkg\vcpkg install opencv

# 开启集成管理
.\vcpkg\vcpkg integrate install

# 生成配置
.\vcpkg\vcpkg integrate project
```

之后需要在 visual studio 中设置 Nuget（Nuget 实际上包平台，类似 npm 之于 nodejs），具体操作可见这篇[博客](https://zhuanlan.zhihu.com/p/153199835)。

设置完成之后，就可以直接`#include<opencv2/opencv.hpp>`而不需要去添加头文件和库目录了。
