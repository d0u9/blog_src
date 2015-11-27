---
title: '更为强大的Vim插件管理器--vim-plug'
categories:
  - Science & Technology
  - Tools
tags:
  - Vim
  - Vim Plugin
  - Tools
abbrlink: 1878247254
date: 2015-11-27 09:44:00
---

之前一直在使用[Vundle](https://github.com/VundleVim/Vundle.vim)作为我的Vim插件
管理器。最近开始使用[Neovim](https://github.com/neovim/neovim)替代了vim，看到
官方主页上推荐了另一款插件－[vim-plug](https://github.com/junegunn/vim-plug).试
用了一下发现真的不错。

> 项目主页：[https://github.com/junegunn/vim-plug](https://github.com/junegunn/vim-plug)

<!-- more -->

# 为何要用vim-plug

因为vim-plug与neovim配合的更好，并且，最重要的是，vim-plug安装插件是并行的。同
时，有报道说，vundle的创始人，主要开发者已经放弃了该项目。

# 如何使用

使用方法与Vundle非常的相似，并且更加的简单。对于Vundle的使用，你可以看看我之前
的文章[VIM插件管理器VUNDLE](http://www.d0u9.xyz/vim-cha-jian-guan-li-qi-vundle/)

## 安装

对于传统vim用户：

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

对于Neovim用户：

```bash
curl -fLo ~/.config/nvim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

## 使用

我们以安装[CtrlP](https://github.com/kien/ctrlp.vim)插件为例子，说明一下
vim-plug的使用方法：

- 在你的`.vimrc`或`nvim/init.vim`中最开始的地方添加如下的内容：

   ```bash
   call plug#begin('~/.vim/plugged')
   
   Plug 'kien/ctrlp.vim'
   
   call plug#end()
   ```

- 打开你的vim或者nvim，执行如下命令：

   ```vim
   :PlugInstall
   ```

- 等待安装完成，重启vim或neovim


# 高级点的使用方法

如果我们只想对C和C++启用YCM插件的话，可以这样配置vim-plug:

```vim
Plug 'Valloric/YouCompleteMe', { 'for': ['c', 'cpp'] }
```

这样YCM只会在我们打开c或者cpp文件的时候才加载了，这样的好处是提高开启速度。

更多用法请参考：[https://github.com/junegunn/vim-plug#on-demand-loading-of-plugins](https://github.com/junegunn/vim-plug#on-demand-loading-of-plugins)

---

### ¶ The end


