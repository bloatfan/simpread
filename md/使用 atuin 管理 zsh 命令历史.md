> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.aloxaf.com](https://www.aloxaf.com/2024/02/manage_zsh_shell_with_atuin/)

> 竟然两年没写博客了，真是自工作以后就越来越懒了。

使用 atuin 管理 zsh 命令历史
--------------------

2024-02-26 · 约 1.1k 字

* * *

竟然两年没写博客了，真是自工作以后就越来越懒了。

不行，我不能再这样颓废下去了，我决定定一个小目标——2024 要写 10 篇博客！

好，现在开始水…… 不对！现在开始写第一篇！

[](#对比-zsh-histdb)对比 zsh-histdb
-------------------------------

atuin 是一个使用 sqlite 来管理 shell 历史记录的工具。

原先一直在用 [zsh-histdb](https://github.com/larkery/zsh-histdb)，这个工具也很好用，唯一的问题没有同步功能。atuin 这点上就做得很棒，这也是吸引我迁移的主要原因。（注：需要启用 sync v2，不然同步逻辑很迷惑）

不过 atuin 也有让我觉得不太好的地方，那就是数据全部放在一个表里，这使得数据库的体积膨胀得很快，因为记录了大量重复的命令和路径。

[](#从-zsh-histdb-迁移)从 zsh-histdb 迁移
-----------------------------------

atuin 提供了从一些常见的历史记录管理插件里迁移的命令。

```
1export HISTDB_FILE=$HISTDB_FILE
2atuin import zsh-hist-db
```

这很好，就是漏了些字段。修完以后随手发了个 PR，被合并以后又发现还有个关于默认值的小问题没修复…… 算了不管了，影响不大，~我自己本地改一下就行~。

[](#配置-atuin)配置 atuin
---------------------

atuin 提供了一份默认配置，可以通过 `atuin init zsh` 获得，但不怎么好用，我得改一下。

### [](#ctrl-r)ctrl-r

我不喜欢 atuin 的界面，花里胡哨，而且还不是异步加载的，在我的 26 万行历史上，体感需要 ~300ms 左右才能打开，而且候选列表很多的时候输入都卡卡的。

甚至还把我的 Up 改了，真是岂有此理。全部删掉！只留下 C-r。

当然 C-r 也要改，就像前面说的我不喜欢 atuin 的界面，直接改成这样：

```
1_atuin_histdb_init() {
2    if (( $+_autin_histdb )); then
3        zsqlite_open -r _autin_histdb ~/.local/share/atuin/history.db
4    fi
5}
8_atuin_search() {
9    emulate -L zsh
10    _atuin_histdb_init
11    local query="
12SELECT DISTINCT command
13FROM history
14WHERE command LIKE ?
15ORDER BY cwd = ? DESC, timestamp DESC
16"
17    local output=$(zsqlite_exec -q _autin_histdb $query ${LBUFFER}% $PWD | ftb-tmux-popup --tiebreak=index --prompt="cmd> " ${LBUFFER:+-q$LBUFFER})
18    if [[ "$output" != "" ]]; then
19        BUFFER=$(echo $output)
20        CURSOR=$#BUFFER
21    fi
22}
```

[zsqlite](https://github.com/Aloxaf/zsh-sqlite) 是我以前为 zsh-histdb 写的一个 zsh 模块，你也可以直接用 sqlite3 命令来查询，但是要注意将参数转义，转义函数参考如下：

```
1sql_escape() {
2    print -r -- ${${@//\'/\'\'}//$'\x00'}
3}
```

`ftb-tmux-popup` 是 fzf-tab 提供的 tmux popup 版 fzf 脚本，非 tmux&fzf-tab 用户直接改成 fzf 就行。

### [](#与-zsh-autosuggestions-结合)与 zsh-autosuggestions 结合

atuin 自带了一个 zsh-autosuggestions 策略，这很好。 但是，默认提供的自动建议只是简单的前缀搜索，这简直是毫无意义……

zsh-autosuggestions 本身就提供了基于历史记录的建议，而主流的 zsh 配置（指 OMZ），都设置了 5 万行内置历史记录大小，用于自动建议绰绰有余。

改掉！

```
1_zsh_autosuggest_strategy_atuin() {
2    emulate -L zsh
3    _atuin_histdb_init
4    local reply=$(zsqlite_exec _autin_histdb "
5SELECT command FROM (
6    SELECT h1.*
7    FROM history h1, history h2
8    WHERE h1.ROWID = h2.ROWID + 1
9        AND h1.session = h2.session
10        AND h2.exit = 0
11        AND h1.command LIKE ?1
12        AND h2.command = ?2
13        AND h1.cwd = ?3
14    ORDER BY timestamp DESC
15    LIMIT 1
16)
17UNION ALL
18SELECT command FROM (
19    SELECT * FROM history WHERE cwd = ?3 AND command LIKE ?1 ORDER BY timestamp DESC LIMIT 1
20)
21UNION ALL
22SELECT command FROM (
23    SELECT * FROM history WHERE command LIKE ?1 ORDER BY timestamp DESC LIMIT 1
24)
25LIMIT 1
26" ${1}% ${history[$((HISTCMD-1))]} $PWD )
27    typeset -g suggestion=$reply
28}
```

这个配置我在 zsh-histdb 里用了很久，迁移到 atuin 上体验也不错（不得不说，只有一个表的话，查询起来确实方便多了），它的逻辑非常简单：

1.  优先执行基于上一条命令的匹配，举例来说，如果在当前目录中曾经出现过这样的历史

```
1> git add -u
2> git commit
```

那么当你第二次输入到这里的时候，它就会直接提示 `git commit`，而不是傻傻地把你的上一条命令又重复一遍。

```
1> git add -u
2> g
```

这是一个受 [mcfly](https://github.com/cantino/mcfly) 启发的配置，虽然它没有 mcfly 那么聪明，有时会给出一些莫名其妙的建议，但整体来讲仍然非常好用，尤其是对于一些关系比较紧密的命令组合，比如 `docker start` 和 `docker attach` 之类的。

2.  如果上一个匹配没有结果，则返回当前目录下，最接近的命令
    
3.  如果上一个匹配仍然没有结果，则直接返回历史记录中最接近的命令
    

本文作者： Aloxaf发布日期： 2024-02-26许可协议： [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)

[#zsh](/tags/zsh) Comments

*   Latest
*   Oldest
*   Hottest

Powered by [Waline](https://github.com/walinejs/waline) v3.8.0