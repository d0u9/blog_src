---
title: 'Vim插件管理器--Vundle'
categories:
  - Science & Technology
  - Tools
tags:
  - Vim
  - Vim Plugin
  - Tools
abbrlink: 3529457901
date: 2015-04-12 05:39:00
---

Vundle（vim & bundle）是一款不错的插件管理器，往其配置文件中添加相应的插件的git源之后，执行命令即可自动安装。这样即使换了机器，也不怕了。

Vundle的Github主页是[https://github.com/gmarik/Vundle.vim](https://github.com/gmarik/Vundle.vim)，下载后按照说明安装即可。

<!-- more -->

# 下载并安装

执行如下代码

```bash
# git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

# 配置.vimrc

将下面的配置文件添加到**.vimrc**文件的最顶端：

```vim
set nocompatible              " be iMproved, required
filetype off                  " required
 
" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')
 
" let Vundle manage Vundle, required
Plugin 'gmarik/Vundle.vim'
 
" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from http://vim-scripts.org/vim/scripts.html
Plugin 'L9'
" Git plugin not hosted on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Avoid a name conflict with L9
Plugin 'user/L9', {'name': 'newL9'}
 
" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList          - list configured plugins
" :PluginInstall(!)    - install (update) plugins
" :PluginSearch(!) foo - search (or refresh cache first) for foo
" :PluginClean(!)      - confirm (or auto-approve) removal of unused plugins
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
```

# 添加插件源
在29行下添加需要用到的插件的git源。这里用supertab举例。

我们通过google可以找到supertab的git项目主页，然后点击下图的位置，复制链接。

![](http://oyui6c341.bkt.clouddn.com/images/2015/vim插件管理器_vundle/01.jpg)

将得到的链接插入到添加到.vimrc的第29行下面，前面需要加上“Bundle”，表示是需要的插件包。

![](http://oyui6c341.bkt.clouddn.com/images/2015/vim插件管理器_vundle/02.jpg)

---

### ¶ The end

