---
title: 'Neovim配置与插件说明'
categories:
  - Science & Technology
  - Tools
tags:
  - Vim
  - Vim Plugin
  - Neovim
  - Tools
abbrlink: 2898109270
date: 2015-11-28 14:05:00
---

由于各种强迫症发作，最近又尝试了一把neovim。说实话，按照我这个量级的使用，neovim的那些个牛逼特性根本没体会出来。。。不过抱着既然是新的，那应该就会不错的心态尝试了一下，发现还好，最后就索性彻底的折腾了一下，顺便尝试着更换了一些插件。

本文主要是想做一个合集，用来介绍我的Neovim的配置，插件和心得。算是一种记录吧，免得以后给别人装的时候忘记了各种误区。

<!-- more -->

# 目录
- [Neovim介绍与安装](#Neovim介绍与安装)
    - [Neovim安装](#Neovim安装)
    - [Neovim的一些改变](#Neovim的一些改变)
    - [Neovim的配置于目录说明](#Neovim的配置于目录说明)
    - [我的插件与配置](#我的插件与配置)
- [使用技巧](#使用技巧)
    - [好用却不常用的快捷键](#好用却不常用的快捷键)
    - [我的按键绑定](#我的按键绑定)
    - [一些有用的命令](#一些有用的命令)
- [Troubleshootings](#Troubleshootings)
    - [与系统剪切板的对接](#与系统剪切板的对接)
    - [Youcompleteme无效](#Youcompleteme无效)
    - [nvim-ipy无效](#nvim-ipy无效)
    - [Ctrl-H 快捷键在OSX下无效](#Ctrl-H-快捷键在OSX下无效)
- 插件介绍
    - [Fugitive](../../posts/2679791153.html)
    - [tagbar](../../posts/2697259906.html#tagbar)
    - [Undotree](../../posts/2697259906.html#Undotree)
    - [ListToggle](../../posts/2697259906.html#ListToggle)
    - [Dash](../../posts/2697259906.html#Dash)
    - [CtrlP](../../posts/953412640.html#CtrlP)
    - [vim-buffergator](../../posts/953412640.html#vim-buffergator)
    - [Ag.vim](../../posts/953412640.html#ag-vim)
    - [Bufferline](../../posts/953412640.html#bufferline)

---

# Neovim介绍与安装

## Neovim安装

> 参考：[https://github.com/neovim/neovim/wiki/Installing-Neovim](https://github.com/neovim/neovim/wiki/Installing-Neovim)

在这里主要说一下Mac OS系统下的安装：

- 安装Homebrew：

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- 安装Neovim：

```bash
brew install neovim/neovim/neovim
```

## Neovim的一些改变

> 参考：[https://neovim.io/doc/user/vim_diff.html#vim-differences](https://neovim.io/doc/user/vim_diff.html#vim-differences)

相对于Vim来说nvim的主要变化体现在了整个架构的改变，大量的采用了异步处理。当然这一 点在平时的使用中是无法体会出来的。除此之外，nvim还移除了很多Vim支持的老旧特性，重构了插件系统。

对于插件来说，大部分的插件可以直接在nvim上使用而不需要修改。不过也有一些插件是不行的，这个需要自己根据使用情况来判断。

具体的可以看参考文献。

## Neovim的配置于目录说明

Vim的配置文件是`~/.vimrc`，插件目录是`~/.vim/`，而nvim的配置文件结构有些类似于emacs。

nvim的使用的[XDG](http://standards.freedesktop.org/basedir-spec/basedir-spec-0.6.html#introduction)标准，简单的说，根配置目录位于`~/.config/nvim/`中。配置文件是`~/.config/nvim/init.vim`，插件目录等依据所用的插件管理器而会不同。

## 我的插件与配置

### 插件列表与说明

- YouCompleteMe

> 主页：[https://github.com/Valloric/YouCompleteMe](https://github.com/Valloric/YouCompleteMe
)

相当牛逼的一款插件，各种语言补全用的。

- vim-airline

> 主页：[https://github.com/bling/vim-airline](https://github.com/bling/vim-airline)

增强你的vim状态栏。

- nerdcommenter

> 主页：[https://github.com/scrooloose/nerdcommenter](https://github.com/scrooloose/nerdcommenter)

一个用来注释各种东西的插件。

- vim-gitgutter

> 主页：[https://github.com/airblade/vim-gitgutter](https://github.com/airblade/vim-gitgutter)

在你的文件旁边显示git diff的到的修改状态。

- tagbar

> 主页：[https://github.com/majutsushi/tagbar](https://github.com/majutsushi/tagbar)

一个用来帮你显示当前文件的类，结构，方法 balabala...

- ctrlp.vim

> 主页：[https://github.com/kien/ctrlp.vim](https://github.com/kien/ctrlp.vim)

用来在多个buffer切换，管理最近打开的文件，管理当前文件夹下的文件。最关键的是其支持模糊匹配。

- vim-buffergator

> 主页：[https://github.com/jeetsukumaran/vim-buffergator](https://github.com/jeetsukumaran/vim-buffergator)

管理vim的buffer。

- vim-bufferline

> 主页：[https://github.com/bling/vim-bufferline](https://github.com/bling/vim-bufferline)

在你的状态栏上显示当前vim的buffer状态。

- nvim-ipy

> 主页：[https://github.com/bfredl/nvim-ipy](https://github.com/bfredl/nvim-ipy)(

这个插件可以让你的vim变成一个python的IDE，依托IPython，可以方便的执行当前编辑的行，或者执行整个文件。


### 下载配置文件

- 我的vim配置可以在我的github上找到[https://github.com/d0u9/.dot](https://github.com/d0u9/.dot).

```bash
git clone https://github.com/d0u9/.dot ~/.dot
mkdir -p ~/.config
```

- 如果你对.dot文件夹里的对其他的内容*没有*兴趣的话再执行下面的命令：

```bash
cp -r ~/.dot/nvim ~/.config/
```

否则，执行：

```bash
ln -s ~/.dot/nvim ~/.config/nvim
```

### 安装插件管理器`vim-plug`

详细见这里[http://www.d0u9.xyz/geng-wei-qiang-de-vimcha-jian-guan-li-qi-vim-plug/](http://www.d0u9.xyz/geng-wei-qiang-de-vimcha-jian-guan-li-qi-vim-plug/)

### 安装插件

- 首先需要安装nvim的python支持，确保你已经安装`pip2`。

```bash
pip2 install neovim
```

- 打开nvim，执行：

```vim
:PlugInstall
```

如果安装完插件后发现有部分插件不能使用，请参考[Troubleshootings](#Troubleshootings)。

- 使用`ag`([the_silver_searcher](https://github.com/ggreer/the_silver_searcher))作为CtrlP的默认搜索工具，ag将会大大加速CtrlP插件的打开速度。如果你没有兴趣的话，请删除`.config/nvim/init.vim`中的`Speed up CtrlP`部分。

> 参考：http://stackoverflow.com/questions/21346068/slow-performance-on-ctrlp-it-doesnt-work-to-ignore-some-folders

安装Ag，这里以OSX举例，其他系统的安装方法请戳[这里](https://github.com/ggreer/the_silver_searcher#installing)：

```bash
brew install the_silver_searcher
```

# 使用技巧

这里推荐一个网站，[http://www.openvim.com/](http://www.openvim.com/)，用这个网站做练习不错。

## 好用却不常用的快捷键

以下的命令都是在Normal模式下：

`b` 移动到上一个单词的开始。

`w` 移动到下一个单词的末尾。

`e` 移动到当前单词的末尾。

`(N) + i + (word) + [ESC]` 插入N个word。e.g. `3igo[ESC]`会插入三个go。

`f + (char)` 向后移动光标到第一个char字母出现的地方。

`F + (char)` 与`f + (char)`相似，但是是往前移动光标。

`(N) + G` 移动光标到第N行。

`:(N)`，和上面的一条一样，也是移动到第N行。

`(` 和 `)` 移动到上一个句子的句首，或下一个句子的句首。

`{` 和 `}` 移动到上一段落的段首，或下一段落的段首。

`%` 移动到`( )` `{ }` `[ ]`括号对的另一半。

`*` 向后查找当前光标所在的单词。

`#` 向前查找当前光标所在的单词。

`zz` 将当前行置于屏幕中间。

`zt` 将当前行置于置于屏幕顶端。

`zb` 将当前行置于置于屏幕底端。

`Ctrl-b` 往上翻页，即PageUp。

`Ctrl-f` 网下翻页，即PageDown。

以下内容是关于window管理的

`ctrl + w` 进入窗口管理，结合其他键管理窗口。

`ctrl + w` `r` 将当前窗口向下或向右移动。

`ctrl + w` `R` 将当前窗口向上或向左移动。

`ctrl + w` `K` 将当前窗口移动到最上边。

`ctrl + w` `J` 将当前窗口移动到最下边。

`ctrl + w` `H` 将当前窗口移动到最左边。

`ctrl + w` `L` 将当前窗口移动到最右边。

`ctrl + w` `_` 将当前窗口最大化。

`ctrl + w` `>` 将当前窗口垂直分割线向右移动。

`ctrl + w` `<` 将当前窗口垂直分割线向左移动。

`ctrl + w` `+` 将当前水平分割的窗口扩大。

`ctrl + w` `-` 将当前水平分割的窗口减小。

> 为了更好快的移动窗口，我映射了一些快捷键。具体见[我的按键绑定](#我的按键绑定)。


## 我的按键绑定 

`,` <leader>键。

`Ctrl + [h|j|k|l]` 在多窗口间快速切换。

`<leader> + /` 反转当前的搜索高亮状态。

`>` or `<` 在选择模式下缩紧或反缩进行。

`w!!` 是不是经常以非root用户编辑root用户权限的文件？这个命令帮助你。

`<leader>ew`, `<leader>ev`, `<leader>es` 打开新文件以新窗口，垂直窗口，水瓶窗口。

`<leader>ff` 打开一个新窗口，里面显示所有当前光标下的单词出现的位置。

`zl`, `zh` 水平滚动窗口。

`<leader>w=` 等宽所有窗口。

`<leader>w'` 增加当前水平分割窗口。

`<leader>w;` 减小当前水平分割窗口。

`<leader>w]` 向左移动垂直分割线。

`<leader>w[` 向右移动垂直分割线。

`<leader>wp` 返回前一次的窗口。


## 一些有用的命令

`:set wrap` 将长的行折断显示。

`:set spell` 拼写检查，写博客，写comment的时候比较有用。

# Troubleshootings

## 与系统剪切板的对接

在你的`init.vim`中添加如下内容：

```vim
set clipboard+=unnamedplus
```

如果你也使用tmux，请安装`reattach-to-user-namespace`，直接用Homebrew安装就可以。

## Youcompleteme无效

如果你安装完插件，发现Youcompleteme无法使用，请进入`.config/nvim/plugged/YouCompleteMe`并执行：

```bash
python ./install.py --clang-completer
```

## nvim-ipy无效

确保安装了`ipykernel`，如果有pip，可通过以下命令安装：

```bash
pip install ipykernel
```

## Ctrl-H 快捷键在OSX下无效

> 参考资料：[https://github.com/neovim/neovim/issues/2048](https://github.com/neovim/neovim/issues/2048)

简而言之，在你的终端下执行如下命令，如果你也用tmux的话，同样需要在tmux环境下也执行如下命令：

```bash
infocmp $TERM | sed 's/kbs=^[hH]/kbs=\\177/' > $TERM.ti
tic $TERM.ti
```

---

### ¶ The end

