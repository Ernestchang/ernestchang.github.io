
title: Vim的常用命令
date: 
categories: 
- linux 

tags:  
- vim

---

 
Vim 指令笔记

vim 的三种模式
```
    - 输入模式：输入内文。
    - 指挥模式：也叫指令模式，主要是进入到可以对文件做修改，复製，剪下贴上，游标移动等动作。
    - 执行模式：文件存档，离开等等行為。
```
常用模式的切换
```
    - 输入模式 -> 指挥模式 : 键盘 Esc
    - 指挥模式 -> 输入模式 : 键盘 i, a, o
    - 指挥模式 -> 执行模式 : 键盘 : 
```
如何进入 vim 编辑器

    - 指令 vim 本身就可以开启 vim 编辑器，跳出请先到执行模式再按 q(:q) 然后 Enter 键。
    - 指令 vim 档名 可开启某档案。

如何离开 vim

    - :q : 直接离开（在文件没有被编辑过的情况下可以用）。
    - :q! : 强制离开（不储存你的修改强制离开）。
    - :wq : 储存并离开 (wq顺序不能反)。
    - :x : 储存并离开。

进入输入模式的几个方式

    - Append:
      - a: 游标前插入文字。
      - A: 游标移到此行最后一个字元。
    - Insert:
      - i: 游标后插入文字。	 	
      - I: 游标移到此行第一个字元。
    - Open a new line:
      - o: 游标所在的那一行，向下插入新的一行。
      - O: 游标所在的那一行，向上插入新的一行。

指挥模式下的常用指令(注意大小写有别)

    - 针对现在画面跳到上中下区域：
      - H : 跳到画面上面。
      - M : 跳到画面中间。
      - L : 跳到画面下方。
    - 上下左右移动：
      - h :	往左移动。
      - j : 往下移动。
      - k : 往上移动。
      - l :往右移动。
      - 数字 ＋ [h, j, k ,l] : 往左,下,右,上移动几个字元。
    - gg : 跳到第一行。
    - G : 跳到最后一行。
    - 数字 ＋ gg : 跳到该数字那一行。
    - 数字 ＋ G : 跳到该数字那一行。
    - b : 移动到上一个字的第一个字元。
    - w : 移动到下一个字的第一个字元。
    - W : 移动到下一个字的第一个字元(以空白键或是tab键当区隔的跳法)。
    - ctrl + f : 下一页（forward）。
    - ctrl + b : 上一页（back）。
    - ctrl + g : 显示你目前位於整份文件的哪一行。
    - ^ : 移到此行的第一个字元。
    - $ : 移到此行的最后一个字元。

指令模式下的操作

    - :q : 直接离开（在文件没有被编辑过的情况下可以用）。
    - :q! : 强制离开（不储存你的修改强制离开）。
    - :wq : 储存并离开 (wq顺序不能反)。
    - :x : 储存并离开。
    - :set nu : 显示行号（也有人说:set number，取消则為:set nonumber）。
    - :set list : 显示看不见的空白字元或tab键。
    - :set hlsearch : 搜寻到的字串反白。 
    - dd : 删除游标所在的那一行。
    - 数字 ＋ dd : 删除游标往下多少行。
    - x: 删除游标后的字元。
    - X: 删除游标前的字元。
    - 数字＋x(X) : 删除游标后（前）多少个字元。
    - u : 复原。
    - ctrl + r : 回复（redo）。
    - yy : 复製游标所在的那一行。
    - 数字 ＋ yy : 复製游标下几行。
    - p : 游标后贴上。
    - P : 游标前贴上。
    - /要搜寻的文字: 游标往下搜寻你要的文字。
    - ?要搜寻的文字 : 游标网上搜寻你要的文字。
    - n : 重复搜寻。
    - N : 反方向的重复搜寻。

c9s teaching

    - \ff 快速找到档案
    - \fb 找到 buffer 中的档案
    - \ff, \fb 搭配 ctrl + (w, P, N, L) 来操作上下左右
    - \ff, \fb 搭配 ctrl + c, **, */*/

Text Object (change inner)

    - c + i + (  
    - - a + {
    - - [
    - - '

视觉选取模式

    - v

补充

    - V 是 line wise selection
    - \ff 的功能是 fuzzyfinder 提供的
    - "\" 通常叫做 leader key



## 打开/退出

    vim -R file1 只读打开
    :qall 退出所有文件
    :wq 写入并退出
    :q! 强制退出

## 插入

    i 在当前位置生前插入
    I 在当前行首插入
    a 在当前位置后插入
    A 在当前行尾插入
    o 在当前行之后插入一行
    O 在当前行之前插入一行

## 移动

    h 左移一个字符
    l 右移一个字符
    k 上移一个字符
    j 下移一个字符

以上四个命令可以配合数字使用，比如20j就是向下移动20行，5h就是向左移动5个字符。

## 删除

    dd 删除当前行
    dj 删除当前行和上一行
    dk 删除当前行和下一行
    10dd 删除当前行开始的共10行
    D 删除当前字符至行尾

## 跳转

    gg 跳转到文件头
    G 跳转到文件尾
    gg=G自动缩进 （非常有用）
    Ctrl + d 向下滚动半屏
    Ctrl + u 向上滚动半屏
    Ctrl + f 向下滚动一屏
    Ctrl + b 向上滚动一屏
    冒号+行号，跳转到指定行；比如:120，跳转到120行；
    $ 跳转到行尾
    0 跳转到行首

## 编辑

    u 撤销
    Ctrl + r 重做
    yy 复制当前行
    按v（逐字）或V（逐行）进入可视模式，然后用jklh命令移动即可选择某些行或字符，再按y即可复制任意部分
    p 粘贴在当前位置
    另外，删除在vim里面就是剪切的意思，所以dd就是剪切当前行，可以用v或V选择特定部分再按d就是任意剪切了

## 查找

    /text　　查找text，按n健查找下一个，按N健查找前一个
    ?text　　查找text，反向查找，按n健查找下一个，按N健查找前一个
    :set ignorecase　　忽略大小写的查找
    :set noignorecase　　不忽略大小写的查找

## 替换

    :s/old/new/ 用old替换new，替换当前行的第一个匹配
    :s/old/new/g 用old替换new，替换当前行的所有匹配
    :%s/old/new/ 用old替换new，替换所有行的第一个匹配
    :%s/old/new/g 用old替换new，替换整个文件的所有匹配
    也可以用v或V选择指定行，然后执行


## 多文件操作

    vim file1 file2 file3 ... 同时编辑多个文件
    :split 将窗口分成上下两个子窗口，对应两个不同的文件
    :vsplit 将窗口分成左右两个子窗口，对应两个不同的文件
    :open file4 打开新文件
    :bn 切换到下一个文件（当前窗口）
    :bp 切换到上一个文件（当前窗口）
    Ctrl-w h    移动到窗口左边
    Ctrl-w j    移动到窗口下边
    Ctrl-w k    移动到窗口上边
    Ctrl-w l    移动到窗口右边


## 报错处理
> 使用vim修改文件报错，系统提示如下：
> E37: No write since last change (add ! to override)

故障原因：
文件为只读文件，无法修改。

解决办法：
使用命令:'w!'强制存盘即可，在vim模式下，键入以下命令：
    :w！
存盘后在使用vim命令检查是否保存，如未保存，编辑后重复以上操作。
