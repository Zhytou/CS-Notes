# CMake

- [CMake](#cmake)
  - [Introduction To CMake Buildsystems](#introduction-to-cmake-buildsystems)
  - [Simple Usage](#simple-usage)
  - [CLI Tool](#cli-tool)
    - [cmake](#cmake-1)
    - [ctest](#ctest)
  - [Advanced Commands](#advanced-commands)
  - [References](#references)

CMake（Cross-platform Make）是一个跨平台的构建自动化工具，可以帮助软件开发者自动生成不同平台下的构建系统文件。此外，CMake不光可以使用Make作为其构建工具，也可以结合Ninja、Visual Studio、QT等使用。

使用CMake，我们可以在一个统一的CMakeLists.txt文件中描述项目的所有构建过程，包括源代码文件、依赖库、编译选项、链接选项等，而不需要为不同的构建系统编写不同的文件。CMake会根据CMakeLists.txt文件自动生成相应的构建系统文件，从而实现跨平台的构建。

## Introduction To CMake Buildsystems

构建系统（BuildSystem）描述了如何使用构建工具从源代码构建项目的可执行文件和库以自动化该过程。 例如，构建系统可以是与GNU Make一起使用的 Makefile文件或Visual Studio的项目文件。

为了避免维护多个此类构建系统，项目可以使用以CMake语言编写的文件抽象地指定其构建系统。要使用CMake生成构建系统，必须指定源码树（Source Tree）和构建树（Build Tree）。其中，前者是指包含项目提供的源文件的顶级目录，后者是指存储构建系统文件和构建输出工件（例如可执行文件和库）的顶级目录。 此外，CMake使用CMakeCache.txt文件来设置构建系统配置选项，并将该目录标识为构建树。

## Simple Usage

**设置**：

CMakeLists.txt是CMake项目的配置文件，包含了项目的设置和指令。CMake读取这个文件来生成相应的构建系统文件，下面展示了一个最简单的CMakeLists.txt。

``` txt
cmake_minimum_required(VERSION 3.0) 
project(Demo)  

add_executable(main main.cpp)
```

值得注意的是，任何CMake项目的顶层CMakeLists.txt都必须首先使用cmake_minimum_required()命令指定最低CMake版本。

**编译**：

由于CMake最主要的目的其实是生成Makefile，所以我们可以先使用CMake生成构建目录中的Makefile文件，再使用make命令编译构建我们想要的目标。比如：

```bash
# 新建构建目录，并且进入该目录
mkdir build; cd build
# 使用cmake生成Makefile
cmake ../
# 使用make构建目标
make all
make main
make clean
```

## CLI Tool

### cmake

cmake是跨平台构建系统生成器CMake的命令行工具。常用的指令包括：

```bash
# 初始化构建目录
# -S指定源文件目录
# -B指定构建目录
cmake -S . -B build
# 进入构建目录，并罗列可构建目标
cd build; make help
# 构建目标
# --build指定已经存在的CMake构建目录（与-B不同，-B是选择初始化位置）
# --target指定需要构建的目标名称
cmake --build . --target xxx
```

### ctest

```bash
# 运行测试
ctest [<options>] [--test-dir <path-to-build>]
# 展示结果
ctest -D <dashboard>         [-- <dashboard-options>...]
ctest -M <model> -T <action> [-- <dashboard-options>...]
ctest -S <script>            [-- <dashboard-options>...]
ctest -SP <script>           [-- <dashboard-options>...]
```

## Advanced Commands

## References

- [CMake Official Tutorial](https://cmake.org/cmake/help/v3.27/guide/tutorial/index.html)
- [CMake系列教程](https://github.com/xizhibei/blog/issues/133)
