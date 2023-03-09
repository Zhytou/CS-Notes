# Node.js Setup

## Install Node.js

前往[官网](https://nodejs.org/zh-cn/download/)，下载相应安装包

## Environment setting

**修改全局缓存和包位置**:

``` bash
# 查看当前全局包位置
npm list -global
# 设置全局包安装位置
npm config set prefix "[nodejs path]\node_global"
# 设置全局缓存位置
npm config set cache "[nodejs path]\node_cache"
```

**配置镜像站地址**：

``` bash
# 设置镜像站地址
npm config set registry=http://registry.npm.taobao.org
# 检查设置是否成功
npm config get registry
```
