# 第 1 章 Shell 入门基础

## 1.4 向脚本传递参数

### 1.4.1 Shell 脚本的参数

变量名 | 说明
---|---
`$n` | 传递给脚本的第 n 个参数，例如 `$1` 表示第 1 个参数，`$2` 表示第 2 个参数……
`$#` | 命令行参数的个数
`$0` | 当前脚本的名称
`$*` | 以“参数1参数2参数3……”的形式返回所有参数的值
`$@` | 以“参数1”“参数2”“参数3”……的形式返回所有参数的值
`$_` | 保存之前执行的命令的最后一个参数

```bash
#-----------------------------/chapter1/ex1-3.sh------------------
#! /bin/bash

echo "$# parameters"
echo "$@"
```

结果：
```
$ sh ex1-3.sh a "b c"
2 parameters
a b c
```

**注意：对于包含空白字符或者其他特殊字符的参数，需要使用单引号或者双引号进行传递。**

如果用户传递的参数多于 9 个，则不能使用 `$10` 来引用第 10 个参数。为了能够获取第 10 个参数的值，用户必须处理或保存第 1 个参数，即 `$1`，然后使用 `shift` 命令删除参数1，并将所有剩余的参数下移 1 位，此时 `$10` 就变成了 `$9`，依此类推。`$#` 的值将被更新以反映参数的剩余数量。

### 1.4.2 参数扩展

```bash
# ex1-4.sh
#!/bin/bash

echo "OPTIND starts at $OPTIND"

# getopts 后的选项字符串前面有冒号 :，编译器不会提示选项错误信息，由用户自己处理。
while getopts ":pq:" optname
do
    case "$optname" in
        "p")
            echo "Option $optname is specified"
            ;;
        "q")
            echo "Option $optname has value $OPTARG"
            ;;
        "?")
            echo "Unknown option $OPTARG" # 自己处理未知选项错误，编译器不提示。
            ;;
        ":")
            echo "No argument value for option $OPTARG" # 自己处理缺少选项参数错误，编译器不提示。
            ;;
        *)
            # Should not occur
            echo "Unknown error while processing options"
            ;;
    esac
    echo "OPTIND is now $OPTIND"
done
```

结果：
```
# 不带参数的选项
$ sh ex1-4.sh -p
OPTIND starts at 1
Option p is specified
OPTIND is now 2

# 选项缺少参数，自己处理错误。
$ sh ex1-4.sh -q
OPTIND starts at 1
No argument value for option q
OPTIND is now 2

# 带参数的选项
$ sh ex1-4.sh -q Hello
OPTIND starts at 1
Option q has value Hello
OPTIND is now 3

# 未知选项，自己处理错误。
$ sh ex1-4.sh -f
OPTIND starts at 1
Unknown option f
OPTIND is now 2
```

```bash
# ex1-4.sh

省略 ...

# getopts 后的选项字符串前面没有冒号 :，编译器会提示选项错误信息。
while getopts "pq:" optname
# while getopts ":pq:" optname

省略 ...

```

结果：
```
$ sh ex1-4.sh -q
OPTIND starts at 1
ex1-4.sh: option requires an argument -- q # 编译器提示缺少选项参数错误
Unknown option 
OPTIND is now 2

$ sh ex1-4.sh -f
OPTIND starts at 1
ex1-4.sh: illegal option -- f # 编译器提示未知选项错误
Unknown option 
OPTIND is now 2
```

#### 其他参考文章：

- [《getopts 命令行参数处理》](http://www.cnblogs.com/xiangzi888/archive/2012/04/03/2430736.html)

使用内部命令 `getopts` 可以很方便地处理命令行参数。一般格式为：

`getopts options variable`

`getopts` 的设计目标是在循环中运行，每次执行循环，`getopts` 就检查下一个命令行参数，并判断它是否合法。即检查参数是否以 `-` 开头，后面跟一个包含在 `options` 中的字母。如果是，就把匹配的选项字母存在指定的变量 `variable` 中，并返回退出状态 `0`；如果 `-` 后面的字母没有包含在 `options` 中，就在 `variable` 中存入一个 `?`，并返回退出状态 `0`；如果命令行中已经没有参数，或者下一个参数不以 `-` 开头，就返回不为 `0` 的退出状态。
```bash
#! /bin/bash

while getopts h:ms option
do
    case "$option" in
        h)
            echo "option: h, value: $OPTARG"
            ;;
        m)
            echo "option: m"
            ;;
        s)
            echo "option: s"
            ;;
        \?)
            echo "Usage: example.sh [-h n] [-m] [-s]"
            exit 1
            ;;
    esac
    echo "Next arg index: $OPTIND"
done

echo "*** Do something now. ***"
```

结果：
```
$ sh example.sh -h 100 -ms
option: h, value: 100
Next arg index: 3
option: m
Next arg index: 3
option: s
Next arg index: 4
*** Do something now. ***

$ sh example.sh -h 100 -m -s
option: h, value: 100
Next arg index: 3
option: m
Next arg index: 4
option: s
Next arg index: 5
*** Do something now. ***
```
注：

- `getopts` 允许把多个选项堆叠在一起（如 -ms）。

- 如果选项后要带参数，必须在对应选项后加冒号 `:`，此时选项和参数之间至少要有一个空白字符分隔，这样的选项不能堆叠。

- 如果在需要参数的选项之后没有找到参数，它就在给定的变量中存入 `?` ，并向标准错误中写入错误消息。否则将实际参数写入特殊变量 `OPTARG`。

- 另外一个特殊变量 `OPTIND`，表示下一个要处理的参数索引，初值是 1，每次执行 `getopts` 时都会更新。
