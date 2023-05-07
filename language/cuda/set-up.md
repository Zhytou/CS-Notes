# CUDA Set-up

## Installation

Install WSL2

Install CUDA on Windows

Install CUDA-Toolkit on WSL

### possible uninstall

way 1:

``` bash
# 运行卸载脚本
cd /usr/local/cuda/bin
sudo ./uninstall_cuda_9.0.pl
# 删除安装文件夹
sudo rm -rf cuda
sudo rm -r cuda-9.0
```

way 2:

``` bash
# 卸载软件及其配置
sudo apt-get --purge remove cuda   
# 卸载软件及其依赖的安装包
sudo apt-get autoremove cuda
# 检查/usr/local/cuda文件夹是否被删除
cd /usr/local         
```

## Hello world

## Reference

- [zhihu win10安装CUDA和cuDNN的正确姿势](https://zhuanlan.zhihu.com/p/94220564)
- [github issue cmake cuda](https://github.com/facebookresearch/faiss/issues/1424)
