---
date: 2022-08-29T12:14:34+08:00
title: "如何设计容器的 entrypoint 🚪"
description: "根据php-fpm-entrypoint学习如何设计容器的入口文件"
tags: ["bash", "docker"]
series: []
featured: true
---


在整理 php-fpm 官方镜像构架方法的时候，注意到了一个入口文件 docker-php-entrypoint， 和 DockerFile 的设计。

为什么要这样设计呢，为镜像制作 entrypoint 的好处是什么的呢？🤔️我们来研究一下

<!--more-->

```docker
ENTRYPOINT ["docker-php-entrypoint"]
CMD ["php-fpm"]
```

先看一下这个 `docker-php-entrypoint` 文件

```bash
#!/bin/sh
set -e

# first arg is `-f` or `--some-option`
if [ "${1#-}" != "$1" ]; then
 set -- php-fpm "$@"
fi

exec "$@"
```

### bash脚本中的set指令

从上到下看，先看一下文件开头的set -e

最常用的两个参数就是 `-e` 与 `-x` ，一般写在 bash 脚本的代码逻辑之前，这两个组合在一起用，可以在 Debug 的时候替你节省许多时间 。

- `set -x` 会在执行每一行 shell 时，把执行的内容输出来。它可以让你看到当前执行的情况，里面涉及的变量也会被替换成实际的值。
- `set -v` 会在执行每一行 shell 时，把执行的内容输出来, 但里面涉及的变量不会被替换。
- `set -e` 会在执行出错时结束程序，就像其他语言中的“抛出异常”一样。
- `set -u` 会在执行 shell 时, 如果遇到不存在的变量会忽略掉这个问题

例如

```bash
#!/bin/bash
# 打印每一行语句的执行情况，执行错误时，则立即退出脚本
# set [-可选参数] [-o 选项]
set -ex
```

当然还有一些参数需要携带 option， 也是常用的设置，例如:

- `set -o pipefail` 对于用管道符`|`连接的命令，如果最后一个命令执行成功，那么这整个命令则会被认为执行成功，大多数情况下是不符合我们需求的。所以设置了这个选项之后，某个子命令出错的时候直接退出。

所以我们在设计一些脚本的时候，可以调加这样的配置让脚本更健壮。

```bash
#!/bin/bash
set -euxo pipefail

...

```

### 修改脚本的输入参数

一般来说，在shell脚本中 `$1,$2,$3,$@` , 用来代表输入的参数

- `$!` ：Shell最后运行的后台 Process 的 PID (后台运行的最后一个进程的进程 ID 号)。
- `$#` ：添加到shell当中参数的个数.
- `$$` ：Shell 本身的 PID (ProcessID，即脚本运行的当前进程 ID 号)。
- `$0` ：脚本本身的文件名.
- `$1` ：传到 Shell 当中的第一个参数。
- `$2` ：传到 Shell 当中的第二个参数。
- `$*` ：所有参数列表。如 `$*` 用" "括起来的情况、以 `$1 $2 … $n` 的形式输出所有参数，此选项参数可超过9个；若不加" "，那么 `$*` 与 `$@` 的输出结果相同
- `$@` ：所有参数列表。如 `$@` 用" "括起来的情况、以 `$1` 、`$2`、 … `$n` 的形式输出所有参数。

所以 php-fpm-entrypoint 的这个 `${1#-}` 其实则是指的是用来，获取第一个参数，并去除掉左面的第一个 `-`。


{{< tabgroup >}}
{{< tab name="eg.1" >}}

```bash
${1#*-} # 去除掉左面第一个-和左面所有字符
```

{{< /tab >}}

{{< tab name="eg.2" >}}

```bash
${1#-*} # 去除掉左面第一个-和右面所有字符
```

{{< /tab >}}

{{< tab name="eg.3" >}}

```bash
${1##-*} # 去除掉左面数最后一个（右面第一个）- 和右面所有字符
```

{{< /tab >}}

{{< tab name="eg.4" >}}

```bash
${1%-*} # 去除掉右面第一个- 和右面所有字符
```

{{< /tab >}}

{{< tab name="eg.5" >}}

```bash
${1%%*-} # 去除掉右面数最后一个一个（左面第一个）- 和左面所有字符
```

{{< /tab >}}

{{< /tabgroup >}}

所以 `${1#-}` 表达的，如果输入的第一个参数不是去掉了左侧第一个的`-`不等于参数本身

可以理解为 如果输入的是一个 `-v` 或者 `—int` 类似参数的话，就会进入if的代码段

最后是`set —-` 这个 `set` 的用法，不同于入口处的 `set` ，这里的含义很有趣：

官方文档中是这样描述的：

[Bash Reference Manual - 4.3.1 The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)

> `--` :
>
> If no arguments follow this option, then the positional parameters are unset. Otherwise, the positional parameters are set to the arguments, even if some of them begin with a '-'.
>

`set --` 如果后续没有 `set --` 参数,则会将全部的参数清空，否则会依次赋值给位置参数 `${1}、${2}...`,即使有些参数以 `-` 开头。

> `-` :
>
> Signal the end of options, cause all remaining arguments to be assigned to the positional parameters. The -x and -v options are turned off. If there are no arguments, the positional parameters remain unchanged.
>

`set -` 如果后续没有 `set -` 参数，**不改变**本身传递的参数，但如果设置了参数，同样会依次赋值给位置参数 `${1}、${2}...` ，不过在使用后使用`set  -xv`不会在打印输出脚本的调试信息，意味着确认输入参数的阶段已经结束

```bash
#!/bin/bash

set -- php-fpm "$@" 
# 用于将 php-fpm 设置在 option 其中的前方
set --  "$@" php  # 在所有参数后插入 php
set -- "-ini" "$@"  # 在所有的参数前放置 -ini
set --  "$1" ok  # 在第一个位后方放置ok ，但是截取后方全部
set --  "$@" ok  # 在所有参数后放置ok
```

我们回到当前的脚本中，发现如果输入的参数是 `-` 开头的话，那么 `$@` 就变为了  `php-fpm` + `$@` 然后 通过`exec “$@”` 被执行。假如输入的是一个其他的完整的命令，或者可执行文件，则不会被替换掉，可以通过 `exec` 直接执行 `$@`。

所以利用 Dockerfile 中 `ENTRYPOINT` 和 `CMD` 的特性，默认镜像会在容器启动的时，启动 php-fpm 服务，但我们也可以替换 `CMD` 传入 `-v` 来实现 替换原本的`CMD[’php-fpm’],` 来通过入口文件获取 ph 的版本。
