---
title: 'Debian下安装YCM及其相关插件'
categories:
  - Science & Technology
  - Tools
tags:
  - Linux
  - Vim
  - Vim Plugin
  - Tools
abbrlink: 4281926687
date: 2015-07-12 11:30:00
---

最近博主又开始折腾了，看到现在用的Vim插件如此的臃肿无序，无法忍受，是时候好好清理下了。

在开始之前请确保已经安装了[Vundle](https://github.com/VundleVim/Vundle.vim)，如果还未安装，请参考[VIM插件管理器VUNDLE](http://test.d0u9.xyz/vim-cha-jian-guan-li-qi-vundle/)。

<!-- more -->

# Vim的支持

首先需要确认你的vim编辑器是否支持python，使用如下命令查看：

```bash
vim --version | grep python
```

如果在输出的结果中能看到`+python`则说明您的vim已支持python，无需再次源码编译。

如果您的vim不支持python，则需从源码重新编译，首先下载最新的vim，下载地址在这里:[http://www.vim.org/download.php](http://www.vim.org/download.php)。

在编译之前，需要安装`ncurses`和`python-dev`：

```bash
sudo apt-get install libncurses5-dev
sudo apt-get install python-dev
```

解压源码包，依次使用如下命令编译：

```bash
./configure --with-features=huge --enable-pythoninterp=yes
make
sudo make install
```

再次使用使用`vim --version`确认是否有`+python`选项。

> 参考：[stackoverflow](http://stackoverflow.com/questions/20160902/how-to-solve-requires-python-2-x-support-in-linux-vim-and-it-have-python-2-6-6) 和 [http://christopherpoole.github.io/setting-up-vim-with-YouCompleteMe/](http://christopherpoole.github.io/setting-up-vim-with-YouCompleteMe/)

# 安装YCM需要的包库

使用包管理器安装以下库或工具:

```bash
build-essential
cmake
git
libclang-3.5-dev
```

# 安装YouCompleteMe

1. 使用Vundle安装YouCompleteMe
2. 编译YouCompleteMe

编译方法如下：

```bash
cd ~/.vim/bundle/YouCompleteMe
./install.sh --clang-completer --system-libclang
```

# 添加配置文件

在`~/.vimrc`中添加：

```vim
let g:ycm_global_ycm_extra_conf = "~/.vim/.ycm_extra_conf.py"
```

新建并编辑`~/.vim/.ycm_extra_conf.py`，在其中填写如下内容：

```vim
let g:ycm_global_ycm_extra_conf = "~/.vim/.ycm_extra_conf.py"
```

```python
# Partially stolen from https://bitbucket.org/mblum/libgp/src/2537ea7329ef/.ycm_extra_conf.py
import os
import ycm_core

# These are the compilation flags that will be used in case there's no
# compilation database set (by default, one is not set).
# CHANGE THIS LIST OF FLAGS. YES, THIS IS THE DROID YOU HAVE BEEN LOOKING FOR.
flags = [
    '-Wall',
    '-Wextra',
    '-Werror',
    '-Wc++98-compat',
    '-Wno-long-long',
    '-Wno-variadic-macros',
    '-fexceptions',
    # THIS IS IMPORTANT! Without a "-std=<something>" flag, clang won't know which
    # language to use when compiling headers. So it will guess. Badly. So C++
    # headers will be compiled as C headers. You don't want that so ALWAYS specify
    # a "-std=<something>".
    # For a C project, you would set this to something like 'c99' instead of
    # 'c++11'.
    '-std=c++11',
    # ...and the same thing goes for the magic -x option which specifies the
    # language that the files to be compiled are written in. This is mostly
    # relevant for c++ headers.
    # For a C project, you would set this to 'c' instead of 'c++'.
    '-x', 'c++',
    # This path will only work on OS X, but extra paths that don't exist are not
    # harmful
    '-isystem', '/System/Library/Frameworks/Python.framework/Headers',
    '-isystem', '/usr/local/include',
    '-isystem', '/usr/local/include/eigen3',
    '-I', 'include'
    '-I.'
]

# Set this to the absolute path to the folder (NOT the file!) containing the
# compile_commands.json file to use that instead of 'flags'. See here for
# more details: http://clang.llvm.org/docs/JSONCompilationDatabase.html
#
# Most projects will NOT need to set this to anything; you can just change the
# 'flags' list of compilation flags. Notice that YCM itself uses that approach.
compilation_database_folder = ''

if compilation_database_folder:
  database = ycm_core.CompilationDatabase( compilation_database_folder )
else:
  database = None


def DirectoryOfThisScript():
  return os.path.dirname( os.path.abspath( __file__ ) )


def MakeRelativePathsInFlagsAbsolute( flags, working_directory ):
  if not working_directory:
    return list( flags )
  new_flags = []
  make_next_absolute = False
  path_flags = [ '-isystem', '-I', '-iquote', '--sysroot=' ]
  for flag in flags:
    new_flag = flag

    if make_next_absolute:
      make_next_absolute = False
      if not flag.startswith( '/' ):
        new_flag = os.path.join( working_directory, flag )

    for path_flag in path_flags:
      if flag == path_flag:
        make_next_absolute = True
        break

      if flag.startswith( path_flag ):
        path = flag[ len( path_flag ): ]
        new_flag = path_flag + os.path.join( working_directory, path )
        break

    if new_flag:
      new_flags.append( new_flag )
  return new_flags


def FlagsForFile( filename ):
  if database:
    # Bear in mind that compilation_info.compiler_flags_ does NOT return a
    # python list, but a "list-like" StringVec object
    compilation_info = database.GetCompilationInfoForFile( filename )
    final_flags = MakeRelativePathsInFlagsAbsolute(
      compilation_info.compiler_flags_,
      compilation_info.compiler_working_dir_ )
  else:
    relative_to = DirectoryOfThisScript()
    final_flags = MakeRelativePathsInFlagsAbsolute( flags, relative_to )

  return {
    'flags': final_flags,
    'do_cache': True
  }
```

---

### ¶ The end
