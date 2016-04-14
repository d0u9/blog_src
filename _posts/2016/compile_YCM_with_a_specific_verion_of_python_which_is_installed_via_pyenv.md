---
title: 'Compile YCM with a specific verion of python which is installed via pyenv'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - Python
abbrlink: 1936695139
date: 2016-04-14 19:32:00
---

# Install python with pyenv

To compile YCM, the python must be installed with `--enable-shared` option enabled. By default, pyenv doesn't install python with this option on, so you need to tell pyenv that we need it:

```bash
CONFIGURE_OPTS="--enable-shared --with-system-expat --with-system-ffi" pyenv install 2.7.13
```

<!-- more -->

# Install and Compile YCM

1.  Install YCM with [vim-plug](https://github.com/junegunn/vim-plug) (or [Vundle](https://github.com/VundleVim/Vundle.vim#about)). I prefer to use `vim-plug`, simply add the following line to your `vimrc` file or `init.vim` file if you use neovim instead.

2.  Before compiling you have to install `cmake`.

3.  Here, we assume you installed YCM with `vim-plug`, the default YCM directory is in `~/.config/nvim/plugged/YouCompleteMe` for neovim users.

   Create new folders for building YCM.

   ```bash
   cd /tmp
   mkdir ycm_build
   mkdir -p ycm_temp/llvm_root_dir
   ```

   For support C-family languages autocomplete, you have to download the binary distribution of LLVM+Clang from [llvm.org](http://llvm.org/releases/download.html), and extract to `~/ycm_temp/llvm_root_dir`

   Then run the following command in `~/ycm_build` directory:

   ```bash
   cmake -G "Unix Makefiles" -DUSE_PYTHON2=OFF                          \
     -DPYTHON_INCLUDE_DIR=~/.pyenv/versions/3.6.0/include/python3.6m    \
     -DPYTHON_LIBRARY=~/.pyenv/versions/3.6.0/lib/libpython3.6m.so      \
     -DPATH_TO_LLVM_ROOT=/usr .                                         \
     ~/.config/nvim/plugged/YouCompleteMe/third_party/ycmd/cpp

   cmake --build . --target ycm_core --config Release
   ```

> Note: Use the latest version of cmake to bypass potential errors or abnormalities.

---

### ¶ The end
