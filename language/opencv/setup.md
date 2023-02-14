# OpenCV Setup

- [OpenCV Setup](#opencv-setup)
  - [Use OpenCV on WSL](#use-opencv-on-wsl)
    - [Install OpenCV](#install-opencv)
    - [Install VcXsrv](#install-vcxsrv)
  - [Use OpenCV on Windows](#use-opencv-on-windows)
  - [Install OpenCV](#install-opencv-1)
  - [Use OpenCV in Visual Studio](#use-opencv-in-visual-studio)

## Use OpenCV on WSL

> WSL + OpenCV + VcXsrv + VSCode
> 关于WSL以及终端的设置可以参考[这里](https://github.com/Zhytou/CS-Notes/blob/main/tool/terminal/setup.md)

### Install OpenCV

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

## Install OpenCV

- 前往[官网](https://opencv.org/releases/)下载安装包；
- 双击运行安装。

## Use OpenCV in Visual Studio

- 在项目-属性-VC++包含目录-包含目录中添加OpenCV头文件路径；

- 在项目-属性-VC++包含目录-库目录中添加OpenCV库路径；

- 在项目-属性-链接器-输入-附加依赖项中添加OpenCV库名称。
