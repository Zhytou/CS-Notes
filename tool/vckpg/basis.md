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
.\vcpkg\vcpkg integrate install
```

## References

[vckpg 官网](https://github.com/microsoft/vcpkg/blob/master/README_zh_CN.md)
