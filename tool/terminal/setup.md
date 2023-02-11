# 终端设置

> Windows terminal + WSL + Zsh + Oh-my-zsh

## WSL

Microsoft商店中下载。之后需要分别在Windows和bios中设置启用Windows的Linux子系统和启用虚拟化技术。

## Windows Terminal

Microsoft商店中下载。

## Zsh

## Oh-my-zsh

**简介**：

Oh My Zsh 是一款社区驱动的命令行工具，正如它的主页上说的，Oh My Zsh 是一种生活方式。 它基于zsh 命令行，提供了主题配置，插件机制，已经内置的便捷操作。 给我们一种全新的方式使用命令行。 Oh My Zsh 是基于zsh 命令行的一个扩展工具集，提供了丰富的扩展功能。

**安装**：

``` bash
sudo apt-get install oh-my-zsh
```

## Powerlevel10k

**安装**：

``` bash
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```
