---
title: 'Vim插件介绍--CtrlP, vim-buffergator, bufferline, ag'
categories:
  - Science & Technology
  - Tools
tags:
  - Vim
  - Vim Plugin
  - Neovim
  - Tools
abbrlink: 953412640
date: 2015-12-17 18:29:00
---

# CtrlP

模糊查询buffer、文件以及MRU(Most recently used)文件。

> 项目主页：[https://github.com/ctrlpvim/ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim)
> 介绍页面：[http://ctrlpvim.github.io/ctrlp.vim/#installation](http://ctrlpvim.github.io/ctrlp.vim/#installation)

- 打开CtrlP。

   `<leader>p` 打开CtrlP。

   `<leader>bb` 打开CtrlP的buffer模式。

   `<leader>bm` 打开CtrlP的Mixm模式。

   `<leader>bs` 打开CtrlP的MRU模式。

<!-- more -->

- 打开后可以进行的操作。

   `<C-b>` 和 `<C-f>` 往后和往前在不同的模式间切换滚动。

   `<C-d>` 只匹配文件名，而不是全路径。

   `<C-r>` 切换到正则表达式模式。

   `<C-j>` `<C-k>` 在搜索结果间进行选择。

   `<C-t>` `<C-v>` <C-s`> 在新的tab，或横竖分屏打开当前文件。

   `<C-y>` 创建新文件。

   `<C-z>` 选择多个文件，之后可以用`<C-o>`打开。

# vim-buffergator

> 项目主页：[https://github.com/jeetsukumaran/vim-buffergator](https://github.com/jeetsukumaran/vim-buffergator)

`<leader>bl` 列出当前所有的buffer。

`<leader>bj` 前一个buffer

`<leader>bk` 下一个buffer

`<leader>bt` 打开一个新的空白buffer。

`<leader>bq` 关闭当前buffer，并返回前一个。

在buffer list界面下

`<C-t>` `<C-v>` `<C-s>` 在新的tab，或横竖分屏打开当前buffer。

# ag.vim

一个使用`ag`命令搜索的插件。在使用之前请确保已经安装了ag(the_silver_searcher)。

> 项目主页：[https://github.com/rking/ag.vim](https://github.com/rking/ag.vim)
> Ag项目主页：[https://github.com/ggreer/the_silver_searcher](https://github.com/ggreer/the_silver_searcher)

在Ag打开的quickfix中，可以使用如下快捷键：

`e`    to open file and close the quickfix window

`o`    to open (same as enter)

`go`   to preview file (open but maintain focus on ag.vim results)

`t`    to open in new tab

`T`    to open in new tab silently

`h`    to open in horizontal split

`H`    to open in horizontal split silently

`v`    to open in vertical split

`gv`   to open in vertical split silently

# bufferline

在状态栏上显示当前已经打开的buffer。

> 项目主页：[https://github.com/bling/vim-bufferline](https://github.com/bling/vim-bufferline)

---

### ¶ The end
