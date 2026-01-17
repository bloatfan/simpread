> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.aloxaf.com](https://www.aloxaf.com/2020/03/fzf-tab/)

> 2020-03-14 · zsh · 约 1.8k 字

2020-03-14 · [zsh](https://www.aloxaf.com/categories/zsh) · 约 1.8k 字

* * *

[![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/293849.svg)](https://asciinema.org/a/293849)

这是一个我念叨了很久的想法，自从了解到 fzf 以后，我就一直在思考这个问题：能不用使用 fzf 作为 zsh 的补全选择菜单？

zsh 默认的补全选择菜单非常难用，当然一般人都会进行配置。比较常用的配置是：

```
1zstyle ':completion:*' menu yes select
```

这样可以使用 Tab 来滚动选择补全。然而它其实还可以更强：

```
1zstyle ':completion:*' menu yes select search
```

这会允许你在列表中搜索。相对来说更好用了一点，但是不支持模糊搜索，而且反应有点慢，仍然不够完美。

要想追求（我眼中的）完美的补全体验，还是得上 fzf。基本思路应该是：截获补全列表、用 fzf 展示并选择、将选择项传回去。但这就面临这一个很严肃的问题：如何截获本该传给 zsh compsys 的补全候选项呢？

作为一个 zsh 萌新，我完全无从下手。不过几经搜索之后我幸运地找到了 [https://github.com/Valodim/zsh-capture-completion](https://github.com/Valodim/zsh-capture-completion) 这个可以捕获 zsh 补全列表的项目，它的思路非常有趣——用 zpty 开启一个新的 shell，然后在这个 shell 里面替换掉一个关键内置函数 `compadd`，捕获本来应该传给 zsh 的补全候选项并输出它们，再 zpty 捕获这些输出。

zsh 里面竟然可以覆盖掉内置函数？？而且原来所有的补全相关的辅助函数都是基于 `compadd` 的？？我果然还是太 naive 了……

参考这个思路，我完成了 [https://github.com/Aloxaf/fzf-tab](https://github.com/Aloxaf/fzf-tab) 。安装方式见 [README.md](https://github.com/Aloxaf/fzf-tab/blob/master/README_CN.md)

这玩意儿几乎地完美实现了我的需求，而且还有附带了一些额外功能：

1.  最基本的：使用 fzf 来模糊搜索补全候选项，而且几乎适用所有场景的补全
2.  还可以使用 Ctrl+Space 来一次选择多个候选项
3.  在补全目录时，还可以使用 / 来快速激活下一次补全。在补全长路径时这个功能尤其有用。
4.  在 tmux 3.2 及以上版本运行时，可以[使用 popup 来展示补全结果](https://asciinema.org/a/367471)
5.  可以使用 F1/F2 来切换不同的分组（这个键位是为了避免冲突随便设置的，实际使用中我自己用的是 ,/.）

结合 fzf 的 `--preview` 开关，还能实现更丰富的功能：

```
1# kill 结束进程时时提供预览
2zstyle ':completion:*:*:*:*:processes' command "ps -u $USER -o pid,user,comm,cmd -w -w"
3zstyle ':fzf-tab:complete:kill:argument-rest' fzf-preview 'ps --pid=$word -o cmd --no-headers -w -w'
4zstyle ':fzf-tab:complete:kill:argument-rest' fzf-flags '--preview-window=down:3:wrap'
6# cd 时在右侧预览目录内容
7zstyle ':fzf-tab:complete:cd:*' fzf-preview 'exa -1 --color=always $realpath'
```

当然，这玩意儿也有不完美的地方。比如：尚不支持 `_approximate` completer、基于 compctl（zsh 的旧补全系统） 的补全也不支持、作为一个外来的 “前端” 与 zsh 本身的结合仍然不是很理想……

不过以绝大多数人对 zsh 的配置程度，我觉得 fzf-tab 应该能满足至少 95% 的用户的需求。

由于是第一次编写 zsh 插件，比较生疏。所幸得到了 powerlevel10k 的作者 romkatv 的帮助，学到了不少知识：

1.  写 zsh 插件和写 zsh 脚本的思路是不同的。虽然两者并没有本质上的区别……

写插件的时候，很关注的一个点是速度——你肯定不希望按下 Tab 以后过了两三秒才弹出补全菜单。

但这并不意味着这个插件就得用 C/C++/Rust 来写——因为 zsh 插件需要处理的数据量通常都非常小。比如 fzf-tab，需要处理的补全候选项一般也就几十上百项。在这个数据规模下，创建进程的开销可能比处理数据的开销更大（尤其是 Windows 上）。

因此，尽量避免创建新进程是写 zsh 插件时需要注意的一点。而且 zsh 本身提供了丰富的功能，绝大多数时候，依靠 zsh 本身的功能都能够完成需求。当然，有时这意味着更长的代码，实际中需要根据数据量权衡该使用哪种方式。

写脚本的话，则是怎么爽怎么来。`cat | grep | cut | awk | sort` 一条龙，管他有几条数据（

2.  要想写出在任何配置下都能正常使用的插件（或者说函数）是有不小难度的……

zsh 是一个非常灵活的 shell，甚至提供了 sh、ksh、csh 的兼容模式！（注意：这个列表中不包含 bash！！）

而且还有大量的开关 `setopt xxx` 可以单独调整 zsh 的行为，比如 `setopt ksh_arrays` 会启用 ksh 风格的数组，此时数组下标会从 0 开始，而且必须使用大括号包裹（也就是 bash 采用的风格）。而且 alias 也会应用到函数中。总之你永远猜不到你的用户使用了什么奇怪的配置……

乍一看要想写出能适用所有配置的插件简直是不可能的，所幸 zsh 提供了 `emulate` 命令，这个命令可以用于指定当前 shell OR 函数的模式（sh、ksh、csh、zsh），在这个过程中，所有的开关会被重置到对应模式的默认值，这就保证了你的函数可以在一个可预测的配置下运行。

但是！！如果某个用户丧心病狂地 `alias emulate=echo` 了的话，你的 `emulate` 会直接执行失败，因此你又不得不用引号来包裹 `emulate` 来防止它被识别为 alias。然后再启用 `no_aliases` 开关来禁止 alias 展开。

你以为这就结束了吗？不！zsh 函数和变量，默认都是全局的！

变量还好，可以通过 `local` 来指定必须是局部变量（不过不少人会漏掉 `i` 这种循环变量）。然而并没有办法来定义局部函数。因此，为了防止覆盖掉 zsh 的内部函数 OR 其他插件定义的函数，插件的函数名不得不带上又丑又长的前缀…… 全局变量也是尽可能丑一点长一点。

前几天就遇到了一个因为全局变量名太普通而导致冲突的真实例子：[https://github.com/agkozak/zsh-z/issues/17](https://github.com/agkozak/zsh-z/issues/17)

学姐的 zsh-z 出现了一个莫名奇妙的报错，经过排查，原因是 `zpm-zsh/colors` 定义了一个名为 `c` 的全局变量。然后 `zsh-z` 里有这么一段代码 `if (( $+ops[-c] ))`，这里出现了一个拼写错误（opts -> ops），所以 ops 会是一个未定义变量、而不是关联数组。zsh 尝试将 ops 理解为普通数组、 `-c` 解释为一个表达式并进行求值，然而 `c` 里面包含了特殊字符，于是求值显然失败了，导致了奇怪的报错……

总而言之，要想写一个完美的 zsh 插件是非常心累的…… 要么写一堆样板代码来创造一个确定的环境、要么祈祷遇到的用户是正常人。

*   [https://github.com/lincheney/fzf-tab-completion](https://github.com/lincheney/fzf-tab-completion) 和 fzf-tab 的功能相似，不过我写 fzf-tab 时没有搜到这个项目，不然我就不会写了……
*   [https://github.com/relastle/pmy](https://github.com/relastle/pmy) 尝试基于 fzf 打造另一个补全后端

本文作者： Aloxaf

发布日期： 2020-03-14