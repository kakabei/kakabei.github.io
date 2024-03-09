---
layout: post
title: linux vim 配置记录
date: 2021-07-02 20:21:12
categories: tools 
tags:   工作提效 
excerpt: vim 编辑器的一些配置记录，新环境的下的配置与查询
---


# 一、PowerVim 

github : [https://github.com/youngyangyang04/PowerVim](https://github.com/youngyangyang04/PowerVim)

如果下载不了，可以在码云上找一下：

gitee : [https://gitee.com/ChenBlue/PowerVim](https://gitee.com/ChenBlue/PowerVim)

## 安装方式

```sh
git clone https://github.com/youngyangyang04/PowerVim.git
cd PowerVim
sh install.sh

```

## 用法

```text
正常模式下的快捷键（非插入模式）
;n              // 打开文件目录树显示在屏幕左侧
;m              // 打开当前函数和变量目录树显示在屏幕右侧
;h              // 光标移动到左窗口 
;l              // 光标移动到右窗口
;k              // 光标移动到上窗口
;j              // 光标移动到下窗口 以上四个快捷键特别是打开多个窗口情况下。使用这个快捷键组合非常实用
;w              // 保存文件
;u              // 向上翻半屏
;d              // 向下翻半屏
;1              // 光标快速移动到行首
;2              // 光标快速移动到行末
;a              // 快速切换.h和cpp文件，写C++的时候很方便
;e              // 打开一个新文件
;z              // 切回shell交互命令，输入fg在切回vim，非常实用
;s              // 水平分屏，并打开文件目录选取想打开的文件，如果想新建文件，;e 就好 
;v              // 竖直分屏，并打开文件目录选取想打开的文件，如果想新建文件，;e 就好 
;fw             // 查找项目内关键字，前提是你的系统已经按照了ACK 
;ff             // 查找项目内文件名 
;gt             // 跳转到变量或者函数定义的地方，前提是安装ctags，并且在在PowerVim输入 ;tg命令 Jump to the definition of the keyword where the cursor is located, but make sure you have make ctags
;gr             // 跳回，对应着;gt
;tg             // 对当前目录打ctag 
;y              // 保存当前选中的目录到系统剪切板，前提是vim支持系统剪切板的寄存器
;gg             // 按顺序光标跳转各个窗口

// 一下快捷键是不用;的，直接在 非插入模式 下输入
e               // 快速删除光标所在的词 
tabc            // 关闭当前tab，可以用:tabnew来打开一个新的tab Close tab, of course you should :tabnew a file first. 
F1              // 编译并运行C++文件，自己写的C++例子的时候一键编译。前提手动在当前目录建一个bin文件夹，这是用来存放编译产生的执行文件 
F1              // 编译Java文件
F2              // 运行Java编译的class文件，一般如果要编译并运行Java文件 按F1编译，在按F2运行
gc              // 快速注释选中的块（是visual模式下选中的块） 
gcc             // 快速当前行
{               // 光标向上移动一个代码块
}               // 光标向下移动一个代码块
di(             // 删除括号里的内容
di{             // 删除花括号里的内容
```

# 二、ctag

安装方式：

```sh
yum install  ctags
```

可以在 vim 中 normal 模式下 执行 `;tg` 也可以在目录下执行 `ctags -R *`。

# 三、ack

安装方式：

```sh 
 sudo yum -y install epel-release ack
```

参考：

1、[常用平台安装 ack](https://wxnacy.com/2018/06/07/install-ack/)

2、[ack!](https://beyondgrep.com/install/)

# 四、vim 一些命令

## 写入、保存、退出

```text
q[uit]                  " 退出
:q!                     " 强制退出
:w[rite]                " 保存
:w!                     " 强制保存，能不能保存成功取决于用户对文件的权限
:w ! sudo tee %         " 如果没有权限保存，试试这个命令
ZZ                      " 两个大写的 Z，没有修改直接退出，有修改保存后退出

:w newfilename          " 另存为新文件
:1, 10 w newfilename    " 将 1 到 10 行的内容另存为新文件
:1, 10 w >> filename    " 将 1 到 10 行的内容另存为新文件
:r filename             " 将目标文件的内容追加到当前光标下一行
:3 r filename           " 将目标文件的内容追加到第 3 行一下

:! ls                   " 暂时离开 Vim 查看当前目录的文件，回车后返回 Vim
```

## 光标移动

```text
h           " 方向键 ←
j           " 方向键 ↓
k           " 方向键 ↑
l           " 方向键 →

0           " 移动到行首
$           " 移动到行尾的回车符
g_          " 移动到行尾最后一个非空字符
gg          " 移动到第一行
G           " 移动到最后一行"

w           " 移动到下一个单词开头
e           " 移动到单词的结尾
b           " 移动到单词的开头

" 不常用
nh          " 向左移动 n 格，n 为数字
nl          " 向右移动 n 格
nj          " 向下移动 n 行
nk          " 向上移动 n 行
n<space>    " 向右移动 n 格，同 nl

H            " 移动到当前屏幕第一行的第一个字符
M            " 移动到当前屏幕中间行的第一个字符
L            " 移动到当前屏幕最后一行的第一个字符
+            " 移动到非空白字符的下一行
-            " 移动到非空白字符的上一行

:n<cr>      " 跳转到第 n 行
```

## 查找与替换

```text
word   " 从光标位置向下搜索 word 单词
?word   " 从光标位置向上搜索 word 单词
n       " 英文字母 n，根据 / 或 ? 搜索的方向定位到下一个匹配目标
N       " 与 n 相反，定位匹配目标

:n1,n2s/word1/word2/g   " n1, n2 表示数字，替换 n1 行到 n2 行的单词
:1,$s/word1/word2/g     " 全文替换，也可以写成 :%s/word1/word2/g
:1,$s/word1/word2/gc    " 全文替换，并出现确认提示
```

## 复制、粘贴、删除

```text
x           " 向后删除一个字符
nx          " 向后删除 n 个字符
X           " 向前删除一个字符
nX          " 向前删除 n 个字符

dd          " 删除当前行
ndd         " 向下删除 n 行
d1G / dgg   " 删除第一行到当前行的数据
dG          " 删除当前行到最后一行的数据
d$          " 删除当前字符到行尾
D           " 删除当前字符到行尾
d0          " 从行首删除到当前字符

yy          " 复制当前行
Y           " 复制当前行
nyy         " 从当前行开始复制 n 行
y1G / ygg   " 从第一行复制到当前行
yG          " 从当前行复制到最后一行
y0          " 从行首复制到当前字符
y$          " 从当前字符复制到行尾

p, P        " 黏贴，p 黏贴到光标下一行，P 黏贴到光标上一行
yyp         " 复制并粘贴
ddp         " 删除并粘贴，相当于下移当前行


"+y         " 复制本文到系统剪切板
"+p         " 粘贴系统剪切板到 Vim（不会影响文本格式）
```

## 插入

```text
i   " 在光标前进入 insert 模式
I   " 在当前行左边第一个非空字符前进入 insert 模式，类似其他编辑器的 <c-a> 快捷键
a   " 在光标后进入 insert 模式
A   " 在当前行右边第一个非空字符前进入 insert 模式，类似其他编辑器的 <c-e> 快捷键
o   " 在光标的下一行插入
O   " 在光标的上一行插入
s   " 删除当前字符，并进入 insert 模式
S   " 删除当前行，并进入插入模式

vc  " 删除当前字符，并进入 insert 模式
cc  " 删除当前行，并进入插入模式
c0  " 删除光标位置到行首，并进入 insert 模式
cg_ " 删除光标位置到行尾最后一个非空字符，并进入 insert 模式
ce  " 删除光标位置到单词末尾，并进入 insert 模式
cw  " 删除光标位置到单词末尾，并进入 insert 模式
ciw " 删除当前单词，并进入 insert 模式
cip " 删除整个段落，并进入 insert 模式
ci( " 删除当前括号内的内容，并进入 insert 模式 适用于 ([{<'` 等所有成对的标签
```

## 撤销重做 

```text
u       " 撤销
<c-r>   " 重做
.       " 重复完成操作
```
## 大小写

```text
~       " 当前字符大小写反转
g~~     " 正行字符大小写反转
vu      " 当前字符小写
vU      " 当前字符大写
vU      " 当前字符大写
viwu    " 当前字符小写
viwU    " 当前字符大写
ggguG   " 文本所有字符小写
gggUG   " 文本所有字符大写

:%s/\<./\u&/g       " 将所有单词首字母大写（需要使用 :nohls 去掉高亮）
:%s/\<./\l&/g       " 将所有单词首字母小写
:%s/.*/\u&          " 将每行第一个字母大写
:%s/.*/\l&          " 将每行第一个字母小写
```

# 打开文件警告

我们在用 Vim 编辑文档事，它会自动在文件同目录下生一个 .filename.swp 文件，它是作为一个缓存存在的，记录了你最后的编辑时刻，如果正常关闭，则会自动删除，如果非常关闭，它就会保存下来。

会出现的告警：

```text
[O]pen Read-Only, (E)dit anyway, (R)ecover, (Q)uit, (A)bort:
```

说明：

- `[O]pen Read-Only` 我不管，继续打开文件。此时文件是只读的，这种情况我们可以使用上边讲到的强制保存来写入文件，但是如果这是其他人正在打开的文件，看看就退出好了。
- `(E)dit anyway`依然不管，继续修改文件。此次修改过后需要记得删除 `.swp` 文件，不然下次还会出现。
- `(R)ecover` 使用之前的缓存进行覆盖。同样 `.swp` 文件要手动删除。
- `(Q)uit Ok`，知道有问题，我退出。然后手动删除 `.swp` 文件再进来，一般都会这样操作。
- `(A)bort` 跟 q 一个意思，也是退出。
- `(D)elete it` 多出来这个操作，是在异常退出 `Terminal` 后出现，你肯定会有机会碰到。它就是直接删除 `.swp` 的意思，只要你确保缓存没有用了。

