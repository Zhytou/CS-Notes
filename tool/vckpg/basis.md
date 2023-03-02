# vckpg

> vckpg 一款 C++的包管理工具，类似 pip 之于 python

## Introduction

和 CMake 的关系

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

## References

[vckpg 官网](https://github.com/microsoft/vcpkg/blob/master/README_zh_CN.md)
