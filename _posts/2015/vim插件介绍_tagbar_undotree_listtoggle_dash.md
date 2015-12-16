---
title: 'Vim插件介绍--tagbar, undotree, ListToggle, Dash'
categories:
  - Science & Technology
  - Tools
tags:
  - Vim
  - Vim Plugin
  - Neovim
  - Tools
abbrlink: 2697259906
date: 2015-12-16 14:47:00
---

# tagbar

用来显示代码中的类，成员，函数等名称，并可做跳转。

> 项目主页：[https://github.com/majutsushi/tagbar](https://github.com/majutsushi/tagbar)

需要注意的是，tarbar插件需要Exuberant Ctags支持。

- 设置快速切换快捷键，我的默认设置为<Ctrl-T>组合键，你可以通过下面的语句修改：

   ```vim
   nnoremap <silent> <leader>tb :TagbarToggle<CR>
   ```

- 使用`s`键在字母排序，和文件位置排序间切换

<!-- more -->

# Undotree

用可视化的树形结构显示vim的历史操作。

> 项目主页：[https://github.com/mbbill/undotree](https://github.com/mbbill/undotree)

- 设置显示undotree的快捷键，我的默认为<leader>tr，你可以通过如下设置：

   ```vim
   let g:undotree_WindowLayout = 2
   ```

- 在Undotree窗口中使用`?`键查看更多帮助。

# ListToggle

快速显示和关闭Vim的locationlist和quickfix窗口的插件：

> 项目主页：[https://github.com/Valloric/ListToggle](https://github.com/Valloric/ListToggle)

- 使用如下的命令来修改快捷键：

   ```vim
   let g:lt_location_list_toggle_map = '<leader>l'
   let g:lt_quickfix_list_toggle_map = '<leader>q'
   ```

# Dash

在vim中直接启动Dash并查询相关的手册。

> 项目主页：[https://github.com/rizzatti/dash.vim](https://github.com/rizzatti/dash.vim)

Examples:

- `:Dash`

   Will search for the word under the cursor in the docset corresponding to the current filetype.

- `:Dash printf`:

   Will search for the word 'printf' in the docset corresponding to the current filetype.

- `:Dash setTimeout javascript`

   Will search for the word 'setTimeout' in the 'javascript' docset.

   Tip: You can use <Tab> to complete the keyword names.

- `:Dash!`

   Will search for the word under the cursor in all docsets (globally).

- `:Dash! func`

   Will search for 'func' in all docsets.

---

### ¶ The end
