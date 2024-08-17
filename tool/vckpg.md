# vckpg

vcpkg是一个开源的C++跨平台包管理器，用于获取和构建各种C++第三方库。类似pip之于python，npm之于nodejs，vcpkg可以帮助开发人员方便地获取和管理项目依赖的库，并提供与不同开发平台和工具链的集成支持。

## Basical usage

### Use vckpg on Windows

**Install vckpg**:

```powershell
git clone https://github.com/microsoft/vcpkg
.\vcpkg\bootstrap-vcpkg.bat
```

**Install new package using vckpg**:

```powershell
.\vcpkg\vcpkg install [package name]
```

**Use vckpg in Visual Studio**:

```powershell
# 开启集成管理
.\vcpkg\vcpkg integrate install
# 生成配置
.\vcpkg\vcpkg integrate project
```

之后需要在 visual studio 中设置 Nuget（Nuget 实际上包平台，类似 npm 之于 nodejs），具体操作可见这篇[博客](https://zhuanlan.zhihu.com/p/153199835)。

### CMake Integration

vcpkg提供了与CMake的集成支持。简单来说，它提供了一个CMake集成脚本（vcpkg.cmake），以便在CMake项目中轻松使用vcpkg安装的库。通过在 CMakeLists.txt中引入vcpkg.cmake，可以自动设置正确的库路径和链接选项，从而使CMake能够找到和链接vcpkg安装的库。比如，下面这个例子就是在CMake中使用vcpkg安装的OpenCV。

```cmake
cmake_minimum_required(VERSION 3.0)
project(YourProject)

# 设置 vcpkg 的根目录
set(VCPKG_ROOT "<vcpkg_installation_path>")

# 包含 vcpkg 提供的 CMake 集成脚本
include(${VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake)

# 在项目中查找并使用 OpenCV
find_package(OpenCV REQUIRED)

# 添加你的源码文件
add_executable(YourExecutable main.cpp)

# 链接 OpenCV 库
target_link_libraries(YourExecutable PRIVATE ${OpenCV_LIBS})
```

## References

[vckpg 官网](https://github.com/microsoft/vcpkg/blob/master/README_zh_CN.md)
