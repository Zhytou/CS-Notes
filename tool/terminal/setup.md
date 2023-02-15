# Terminal Setup

- [Terminal Setup](#terminal-setup)
  - [Prettify Shell on WSL](#prettify-shell-on-wsl)
    - [Zsh](#zsh)
    - [Oh-my-zsh](#oh-my-zsh)
    - [Zsh Themes：Powerlevel10k](#zsh-themespowerlevel10k)
  - [Prettify Shell on Windows](#prettify-shell-on-windows)
    - [PowerShell](#powershell)
    - [On-my-posh](#on-my-posh)
    - [PowerShell Themes](#powershell-themes)
  - [Use Windows Terminal to Mange All Your Shells](#use-windows-terminal-to-mange-all-your-shells)
  - [References](#references)

## Prettify Shell on WSL

**简介**：

WSL指的是`Windows Subsystem for Linux`，我们可以在其中运行未经修改过的原生`Linux ELF`可执行文件。

**安装**：

可以在Microsoft商店中下载。之后需要分别在Windows和bios中设置启用Windows的Linux子系统和启用虚拟化技术。

### Zsh

### Oh-my-zsh

**简介**：

`Oh My Zsh`是一款社区驱动的命令行工具，正如它的主页上说的，`Oh My Zsh`是一种生活方式。它基于`zsh`命令行，提供了主题配置，插件机制，已经内置的便捷操作，给我们一种全新的方式使用命令行。

总的来说，`Oh My Zsh`就是基于`zsh shell`的一个扩展工具集，能够帮助你定制你的`zsh shell`。

**安装**：

``` bash
sudo apt-get install oh-my-zsh
```

### Zsh Themes：Powerlevel10k

**安装**：

``` bash
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```

## Prettify Shell on Windows

### PowerShell

### On-my-posh

**安装**：

在Microsoft商店中下载。

**配置**：

``` bash
# 使用vscode编辑启动powershell脚本
code .$PROFILE or notepad .$PROFILE
# 打开后，在该脚本中添加以下指令
 oh-my-posh init pwsh | Invoke-Expression
# 接着保存退出，启用该脚本
. $PROFILE
# 注意：若PowerShell报错不允许执行任何脚本，则需要在管理员模式下执行以下命令后再次启用$PROFILE
set-ExecutionPolicy RemoteSigned
```

### PowerShell Themes

**配置**：

``` bash
# 获取Oh-my-posh主题
Get-PoshThemes
# 选择脚本，修改$PROFILE为以下
 oh-my-posh init pwsh | Invoke-Expression
oh-my-posh init pwsh --config 'C:\Users\[Your Name]\AppData\Local\Programs\oh-my-posh\themes\[Theme Name].omp.json'| Invoke-Expression
# 重新启用$PROFILE
. $PROFILE
```

**其他**：

一般来说，还需要安装字体（可以前往[这里]寻找字体(https://www.nerdfonts.com/)），才能保证显示正常。

## Use Windows Terminal to Mange All Your Shells

`PowerShell`是命令行程序，真正执行指令的程序，而`Windows Terminal`则是管理各种命令行的工具。

Microsoft商店中下载。

## References

- [PowerShell 美化配置](https://blog.csdn.net/qq_33618417/article/details/126856501)
- [PowerShell 脚本禁用解决](https://blog.csdn.net/llf_cloud/article/details/81069099)