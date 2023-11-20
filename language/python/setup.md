# 环境配置

- [环境配置](#环境配置)
  - [包管理](#包管理)
  - [多版本管理](#多版本管理)
    - [Conda](#conda)
    - [Pyenv \& Pipenv](#pyenv--pipenv)

## 包管理

pip是Python标准的包管理工具。使用pip可方便地安装、卸载和更新Python包。它的常见操作包括：

```bash
pip install xxx==8.0.1
pip uninstall xxx
pip list xxx
pip show xxx
pip search xxx
```

**查看下载位置**：

```bash
python -m site
```

**依赖导出**：

``` bash
# 导出全局包，可能存在未使用的包
pip freeze > requirements.txt

# 导出当前项目的依赖
pip install pipreqs
pipreqs .
```

**使用镜像**：

```bash
# 临时使用镜像
pip install xxx -i https://pypi.tuna.tsinghua.edu.cn/simple

# 修改配置文件
vim ~/.pip/pip.conf

[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host = https://pypi.tuna.tsinghua.edu.cn
```

## 多版本管理

Python的版本管理其实分为两部分，一个是Python版本管理，一个是虚拟环境管理（第三方包管理）。

如果我们不使用虚拟环境管理的话，我们所有的包都会安装在系统全局Python（Global Python, 也就是系统默认的Python版本）的site-packages目录下，这会导致两个问题：

- 一是由于在同一个目录下只能安装某一个包的一个版本，如果有两个项目对同一个依赖包有不同的版本需求，就会导致依赖冲突，给开发带来问题。
- 二是哪怕没有依赖冲突的问题，所有的依赖包安装在同一个目录下，会导致导出的依赖文件过于冗余，这在进行项目开源的时候是非常忌讳的。

而进行虚拟环境管理其实也非常简单，类似npm管理javascript包，python也提供了相应工具通过软连接的方式指定特定版本Python，并将第三方包下载到特定目录中，从而达到彼此隔离的目的。

### Conda

Conda也是一款通用的包管理工具，但它同时也兼具版本管理的功能。因为它将一切都视为包，甚至包括它自己。

```bash
# 创建虚拟环境并指定 Python 版本
conda create -n venv_name python=3.11 
# 激活虚拟环境
conda activate venv_name 
# 离开虚拟环境
conda deactivate 
```

### Pyenv & Pipenv

Pyenv用来管理Python版本，而Pipenv则是用来创建虚拟环境。

```bash
# 安装pyenv依赖
sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl
# 安装pyenv
curl https://pyenv.run | bash
# 将pyenv添加至环境变量
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
# 查看可以安装的python 列表
pyenv install -l
# 安装python3.7.6
pyenv install 3.7.6
# 查看电脑中已安装python版本
pyenv versions 
# 设置默认版本Python
pyenv global 3.7.6
```

默认情况下Pipenv会同一管理所有的虚拟环境，把所有的包都放在~/.virtualenvs目录中。如果我们想将虚拟环境放到项目中，我们可以我们可以通过在项目目录下创建一个名为.venv的空目录，之和Pipenv就会自动的将包和相关配置放在项目目录中。

```bash
cd project
mkdir .venv
# 创建虚拟环境
pipenv install

# 安装依赖
pipenv install xxx
# 启动虚拟环境
pipenv shell
# 退出虚拟环境
exit or ctrl+d
```
