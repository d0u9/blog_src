---
title: 'Vim configuration for Linux kernel development'
categories:
  - Science & Technology
  - Tools
tags:
  - Vim
  - Vim Plugin
  - Tools
  - Linux
  - Linux Kernel
abbrlink: 2052776767
date: 2015-11-13 06:39:00
---

I asked this question on [Stackoverflow](http://stackoverflow.com/questions/33676829/vim-configuration-for-linux-kernel-development). Well, even I received the down vote, however, I still feel that the answer from Sam Protsenko gives me a lot help.

<!-- more -->

---

Main differences between Linux kernel and regular C project (from developer's point of view) are next:

 - kernel is very big project (so you should choose which code to index)
 - it has architecture dependent code (and you are only interested in one specific architecture at a time; other architecture shouldn't be indexed)
 - it has very specific [coding style][1] you should stick to (and vim should be configured to display code accordingly)
 - it doesn't use C standard library, but rather has it's own similar routines (so your index tool shouldn't index libc sources)

## Installing indexing tools ##

To navigate kernel code I can propose you to use `cscope` and `ctags` tools. To install them run next command:

    $ sudo aptitude install cscope exuberant-ctags

A little explanation:

 - `cscope`: will be used to navigate the code (switch between functions, etc.)
 - `ctags`: needed for `Tagbar` plugin (will be discussed further) and for `Omni completion` (auto completion mechanism in vim); can be also used for navigation

## Creating index database ##

Now you should index your kernel. That's the tricky part. Insight were taken from [here][2].

First you need to create `cscope.files` file which would list all files you want to index. For example, I'm using next commands to list files for ARM architecture (`arch/arm`), and particularly OMAP platform:

<!-- language: sh -->

    $ find  $dir                                           \
            -path "$dir/arch*"             -prune -o       \
            -path "$dir/tmp*"              -prune -o       \
            -path "$dir/Documentation*"    -prune -o       \
            -path "$dir/scripts*"          -prune -o       \
            -type f -name "*.[chsS]" -print > cscope.files

    $ find  $dir/arch/arm/boot/                            \
            $dir/arch/arm/common/                          \
            $dir/arch/arm/include/                         \
            $dir/arch/arm/kernel/                          \
            $dir/arch/arm/lib/                             \
            $dir/arch/arm/mach-omap2/                      \
            $dir/arch/arm/mm/                              \
            $dir/arch/arm/plat-omap/                       \
            -type f -name "*.[chsS]" -print >> cscope.files

For x86 architecture (`arch/x86`) you can use something like this:

<!-- language: sh -->

    $ find  $dir                                           \
            -path "$dir/arch*"             -prune -o       \
            -path "$dir/tmp*"              -prune -o       \
            -path "$dir/Documentation*"    -prune -o       \
            -path "$dir/scripts*"          -prune -o       \
            -type f -name "*.[chsS]" -print > cscope.files

    $ find  $dir/arch/x86/boot/                            \
            $dir/arch/x86/crypto/                          \
            $dir/arch/x86/ia32/                            \
            $dir/arch/x86/include/                         \
            $dir/arch/x86/kernel/                          \
            $dir/arch/x86/lib/                             \
            $dir/arch/x86/mm/                              \
            $dir/arch/x86/pci/                             \
            $dir/arch/x86/power/                           \
            $dir/arch/x86/realmode/                        \
            $dir/arch/x86/syscalls/                        \
            $dir/arch/x86/video/                           \
            -type f -name "*.[chsS]" -print >> cscope.files

Where `dir` variable can have one of next values:

 - `.`: if you are gonna work only in kernel source code directory; in this case those commands should be run from root directory of kernel source code
 - **absolute path to your kernel source code directory**: if you are gonna develop some out-of-tree kernel module; it this case script can be run from anywhere

I'm using first option (`dir=.`), because I'm not developing any out-of-tree modules.

Now when `cscope.files` file is ready, we need to run actual indexing:

    $ cscope -b -q -k

Where `-k` parameter tells `cscope` to not index C standard library (as kernel doesn't use it).

Now it's time to create `ctags` index database. To accelerate this stage, we're gonna use already created `cscope.files`:

    $ ctags -L cscope.files

Ok, `cscope` and `ctags` index databases are built, and you can remove `cscope.files` file, as we don't need it anymore:

    $ rm -f cscope.files

Next files contain index databases (for `cscope` and `ctags`):

    - cscope.in.out
    - cscope.out
    - cscope.po.out
    - tags

Keep them in root of kernel sources directory.

## vim plugins ##

Next we are gonna install some plugins for vim. To have a better grasp on it, I encourage you to use **pathogen** plugin. It allows you to just `git clone` vim plugins to your `~/.vim/bundle/` and keep them isolated, rather than mixing files from different plugins in `~/.vim` directory.

Install **pathogen** like it's described [here][3].

Don't forget to do next stuff (as it's described at the same link):

>Add this to your `vimrc`:

>     execute pathogen#infect()

> If you're brand new to Vim and lacking a `vimrc`, `vim ~/.vimrc` and paste in the following super-minimal example:

>     execute pathogen#infect()
>     syntax on
>     filetype plugin indent on


## Installing cscope plugin for vim ##

To use cscope in vim, you need to install `cscope_maps.vim` plugin first.
To install it using **pathogen** you can just clone [this][4] repo to your `~/.vim/bundle`:

    $ git clone https://github.com/joe-skb7/cscope-maps.git ~/.vim/bundle/cscope-maps

Now you should be able to navigate between functions and files in vim. Open some kernel source file, put your keyboard cursor on some function call, and press <kbd>Ctrl</kbd>+<kbd>\</kbd> followed by <kbd>g</kbd>. It should bring you to the function implementation. Or it can show you all available function implementations, then you can choose which one to use: [![cscope-struct][5]][5].

For the rest of key mappings see [cscope_maps.vim][6] file.

You can also use commands in vim like:

    :cs f g kmalloc

See `:help cscope` for details.

## ctags note ##

ctags still can be useful for navigation, for example when looking for some `#define` declaration. You can put cursor on this define usage and press <kbd>g</kbd> followed by <kbd>Ctrl</kbd>+<kbd>]</kbd>. See [this answer][7] for details.

## out-of-tree modules development note ##

If you are developing out-of-tree module, you will probably need to load `cscope` and `ctags` databases from your kernel directory. It can be done by next commands in vim (in command mode).

Load external cscope database:

    :cs add /path/to/your/kernel/cscope.out

Load external ctags database:

    :set tags=/path/to/your/kernel/tags

# vimrc #

Some modifications need to be done to your `~/.vimrc` as well, in order to better support kernel development.

First of all, let's highlight 81th column with vertical line (as kernel coding requires that you should keep your lines length at 80 characters max):

    " 80 characters line
    set colorcolumn=81
    "execute "set colorcolumn=" . join(range(81,335), ',')
    highlight ColorColumn ctermbg=Black ctermfg=DarkRed

Uncomment second line if you want to make 80+ columns highlighted as well.

Trailing spaces are prohibited by kernel coding style, so you may want to highlight them:

    " Highlight trailing spaces
    " http://vim.wikia.com/wiki/Highlight_unwanted_spaces
    highlight ExtraWhitespace ctermbg=red guibg=red
    match ExtraWhitespace /\s\+$/
    autocmd BufWinEnter * match ExtraWhitespace /\s\+$/
    autocmd InsertEnter * match ExtraWhitespace /\s\+\%#\@<!$/
    autocmd InsertLeave * match ExtraWhitespace /\s\+$/
    autocmd BufWinLeave * call clearmatches()


## Kernel coding style ##

To make vim respect kernel coding style, you can pull ready to use plugin, like [kernel-coding-style][8] or [vim-linux-coding-style][9].

## Useful plugins ##

Next plugins are commonly used, so you can find them useful as well:

 - [NERDTree][10]
 - [Tagbar][11]
 - [file-line][12]
 - [vim-airline][13]

## Omni completion ##

Auto completion works rather slow on such a big project as kernel. If you still want it, you can add next lines to your `~/.vimrc`:

    " Enable OmniCompletion
    " http://vim.wikia.com/wiki/Omni_completion
    filetype plugin on
    set omnifunc=syntaxcomplete#Complete
    
    " Configure menu behavior
    " http://vim.wikia.com/wiki/VimTip1386
    set completeopt=longest,menuone
    inoremap <expr> <CR> pumvisible() ? "\<C-y>" : "\<C-g>u\<CR>"
    inoremap <expr> <C-n> pumvisible() ? '<C-n>' :
      \ '<C-n><C-r>=pumvisible() ? "\<lt>Down>" : ""<CR>'
    inoremap <expr> <M-,> pumvisible() ? '<C-n>' :
      \ '<C-x><C-o><C-n><C-p><C-r>=pumvisible() ? "\<lt>Down>" : ""<CR>'
    
    " Use Ctrl+Space for omni-completion
    " http://stackoverflow.com/questions/510503/ctrlspace-for-omni-and-keyword-completion-in-vim
    inoremap <expr> <C-Space> pumvisible() \|\| &omnifunc == '' ?
      \ "\<lt>C-n>" :
      \ "\<lt>C-x>\<lt>C-o><c-r>=pumvisible() ?" .
      \ "\"\\<lt>c-n>\\<lt>c-p>\\<lt>c-n>\" :" .
      \ "\" \\<lt>bs>\\<lt>C-n>\"\<CR>"
    imap <C-@> <C-Space>
    
    " Popup menu hightLight Group
    highlight Pmenu ctermbg=13 guibg=LightGray
    highlight PmenuSel ctermbg=7 guibg=DarkBlue guifg=White
    highlight PmenuSbar ctermbg=7 guibg=DarkGray
    highlight PmenuThumb guibg=Black
    
    " Enable global scope search
    let OmniCpp_GlobalScopeSearch = 1
    " Show function parameters
    let OmniCpp_ShowPrototypeInAbbr = 1
    " Show access information in pop-up menu
    let OmniCpp_ShowAccess = 1
    " Auto complete after '.'
    let OmniCpp_MayCompleteDot = 1
    " Auto complete after '->'
    let OmniCpp_MayCompleteArrow = 1
    " Auto complete after '::'
    let OmniCpp_MayCompleteScope = 0
    " Don't select first item in pop-up menu
    let OmniCpp_SelectFirstItem = 0

And use <kbd>Ctrl</kbd>+<kbd>Space</kbd> for auto completion.

## Color schemes ##

First of all you want to be sure that your terminal supports 256 colors. Then put next line to your `~/.vimrc`:

    set t_Co=256

Now download schemes you are prefer to `~/.vim/colors` and select them in `~/.vimrc`:

    colorscheme hybrid
    set background=dark

Which color scheme to use it strongly opinion based matter. I may recommend [mrkn256][14] and [hybrid][15] for starters.


  [1]: https://www.kernel.org/doc/Documentation/CodingStyle
  [2]: http://cscope.sourceforge.net/large_projects.html
  [3]: https://github.com/tpope/vim-pathogen
  [4]: https://github.com/joe-skb7/cscope-maps
  [5]: http://i.stack.imgur.com/mckjb.png
  [6]: https://github.com/joe-skb7/cscope-maps/blob/master/plugin/cscope_maps.vim#L52
  [7]: http://stackoverflow.com/a/1749621/3866447
  [8]: https://github.com/bhilburn/kernel-coding-style
  [9]: https://github.com/vivien/vim-linux-coding-style/commits/master
  [10]: https://github.com/scrooloose/nerdtree
  [11]: https://github.com/majutsushi/tagbar
  [12]: https://github.com/bogado/file-line
  [13]: https://github.com/bling/vim-airline
  [14]: https://github.com/mrkn/mrkn256.vim
  [15]: https://github.com/w0ng/vim-hybrid
