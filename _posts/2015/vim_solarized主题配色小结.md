---
title: 'Vim Solarized主题配色小结'
categories:
  - Science & Technology
  - Tools
tags:
  - Vim
  - Tools
abbrlink: 2932705170
date: 2015-08-03 11:39:00
---

本人对配色特别敏感，在不是自己喜欢的配色环境中工作会严重影响工作效率。之前一直在使用Solarized这款配色。这款配色在各大平台，IDE，终端中都有所支持。不过，本人对其中Vim的配色不太喜欢，总感觉有些单调（本人色弱），因此花了一点点时间来调整使之符合我的要求。

<!-- more -->

# 256颜色支持

首先需要确定是您使用的环境支持256的终端配色，Vim使用256的配色可以方便的和终端配色区分开。这样就不会搞乱终端的配色了。

那么哪些终端支持256颜色呢，只要你使用的终端不是太奇怪的话，都会支持256的配色。

# 256颜色表

256个颜色到底有哪些呢？经过一番查找，我找到了如下图（出处网址[http://www.calmar.ws/vim/256-xterm-24bit-rgb-color-chart.html](http://www.calmar.ws/vim/256-xterm-24bit-rgb-color-chart.html))

![](http://oyui6c341.bkt.clouddn.com/images/2015/vim_solarized主题配色小结/01.png)

# 在哪里自定义颜色

首先你需要下载`Solarized`的Vim配色文件。

下载地址在[这里](https://github.com/altercation/ethanschoonover.com/tree/master/projects/solarized/vim-colors-solarized)。

下载好之后将`colors`文件夹中`solarized.vim`复制到`~/.vim/colors/中`。之后用任意文本编辑器打开`~/.vim/colors/solarized.vim`。跳转到第301行。可见到如下内容：

```vim
elseif g:solarized_termcolors == 256
    let s:vmode       = "cterm"
    let s:base03      = "234"
    let s:base02      = "235"
    let s:base01      = "239"
    let s:base00      = "240"
    let s:base0       = "244"
    let s:base1       = "245"
    let s:base2       = "187"
    let s:base3       = "230"
    let s:yellow      = "136"
    let s:orange      = "166"
    let s:red         = "124"
    let s:magenta     = "125"
    let s:violet      = "61"
    let s:blue        = "33"
    let s:cyan        = "37"
    let s:green       = "64"
```

每一项后面的数字就是[256颜色表](http://www.d0u9.xyz/archives/39.html#256-colors)中的编号。

# 每一项都代表了什么意思

由于本人主要是写C代码，经过了反复的测试实践，我总结的配色项目和C代码的对应关系如下（其中部分没有测出来）：

| 项目名    | 影响内容                                               |
| :-------: | :----------------------------------------------------- |
| base03    | 不详                                                   |
| base02    | :set list :set cursorline :set cursorcolum显示的背景   |
| base01    | 普通注释颜色                                           |
| base00    | :set charlist标记的颜色                                |
| base0     | 函数名，标示符等普通字体颜色                           |
| base1     | vim命令补全时，弹出选择栏的背景色                      |
| base2     | vim命令补全时，弹出选择栏，当前选中项的背景色          |
| base3     | 不详                                                   |
| yellow    | int char struct等变量关键字的颜色                      |
| orange    | #开头的宏定义关键字的颜色                              |
| red       | dxoygen 注释标记的颜色                                 |
| magenta   | 不详                                                   |
| violet    | 不详                                                   |
| blue      | 不详                                                   |
| cyan      | 字符串，include中的文件名的颜色                        |
| green     | 关键字颜色，例如for while return if等                  |

# 我的palette配色

| 颜色  |            |             |              |              |               |               |              |               |
| ----- | ---------- | ----------- | ------------ | ------------ | ------------- | ------------- | ------------ | ------------- |
| RGB   | 46,52,54   | 204,0,0     | 78,154,6     | 196,160,0    | 52,101,164    | 117,80,123    | 6,152,154    | 211,215,207   |
| HEX   | #2E3436    | #CC0000     | #4E9A06      | #C4A000      | #3465A4       | #75507B       | #06989A      | #D3D76B       |
| RGB   | 85,87,83   | 239,41,41   | 138,226,52   | 252,233,79   | 114,159,207   | 173,127,168   | 52,226,226   | 238,238,238   |
| HEX   | #555753    | #EF2929     | #8AE234      | #FCE94F      | #729FCF       | #AD7FA8       | #34E2E2      | #EEEEEE       |

---

### ¶ The end

