# awk 命令的使用

当你第一次拿起双手在电脑上使用 awk 命令处理一个或者多个文件的时候，它会依次读取文件的每一行内容, 然后对其进行处理，awk 命令默认从 stdio 标准输入获取文件内容, awk 使用一对单引号来表示 一些可执行的脚本代码，在可执行脚本代码里面，使用一对花括号来表示一段可执行代码块，可以同时存在多个代码块。 awk 的每个花括号内同时又可以有多个指令，每一个指令用分号分隔，awk 其实就是一个脚本编程语言。说了这么多，你肯定还是一脸的懵逼。你猜对了，上面这些都是废话。先别急，客官请往下看。。。
   
**awk 命令的基本格式**

```
awk [options] 'program' file
```
   
`options` 这个表示一些可选的参数选项，反正就是你爱用不用，不用可以拉到。。。
`program` 这个表示 awk 的可执行脚本代码，这个是必须要有的。
`file`    这个表示 awk 需要处理的文件，注意是纯文本文件，不是你的 mp3，也不是 mp4 啥的。。
   
**先来一个 awk 的使用例子热热身**

```
$ awk '{print $0}' /etc/passwd
```
   
awk 命令的可执行脚本代码使用单引号括起来，紧接着里面是一对花括号，记住是 "花括号" 不是 "花姑娘"，然后花括号里面就是一些可执行的脚本代码段，当 awk 每读取一行之后，它会依次执行双引号里面的每个脚本代码段，在上面这个例子中， `$0` 表示当前行。当你执行了上面的命令之后，它会依次将 /etc/passwd 文件的每一行内容打印输出，你一定在想：这有个毛用，用 cat 命令也能搞定。没错！上面这个命令没个毛用，请往下看。
   
   
**awk 自定义分隔符**

awk 默认的分割符为空格和制表符，我们可以使用 -F 参数来指定分隔符
   
```
$ awk -F ':' '{print $1}' /etc/passwd
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
operator
games
ftp
nobody
```
   
上面的命令将 /etc/passwd 文件中的每一行用冒号 *:* 分割成多个字段，然后用 print 将第 1 列字段的内容打印输出
   
**如何在 awk 中同时指定多个分隔符**
   
比如现在有这样一个文件 some.log 文件内容如下
   
```
Grape(100g)1980
raisins(500g)1990
plum(240g)1997
apricot(180g)2005
nectarine(200g)2008
```
   
现在我们想将上面的 some.log 文件中按照 "水果名称(重量)年份" 来进行分割
   
```
$ awk -F '[()]' '{print $1, $2, $3}' some.log
Grape 100g 1980
raisins 500g 1990
plum 240g 1997
apricot 180g 2005
nectarine 200g 2008
```

在 `-F` 参数中使用一对方括号来指定多个分隔符，awk 处理 some.log 文件时就会使用 "(" 和 ")" 来对文件的每一行进行分割。

**awk 内置变量的使用**

`$0`  这个表示文本处理时的当前行

`$1`  表示文本行被分隔后的第 1 个字段列

`$2`  表示文本行被分割后的第 2 个字段列

`$3`  表示文本行被分割后的第 3 个字段列

`$n`  表示文本行被分割后的第 n 个字段列

`NR`  表示文件中的行号，表示当前是第几行

`NF`  表示文件中的当前行列的个数，类似于 mysql 数据表里面每一条记录有多少个字段

`FS`  表示 awk 的输入分隔符，默认分隔符为空格和制表符，你可以对其进行自定义设置

`OFS` 表示 awk 的输出分隔符，默认为空格，你也可以对其进行自定义设置

`FILENAME` 表示当前文件的文件名称，如果同时处理多个文件，它也表示当前文件名称
   
比如我们有这么一个文本文件 fruit.txt 内容如下，我将用它来向你演示如何使用 awk 命令工具，顺便活跃一下此时尴尬的气氛。。

```
peach    100   Mar  1997   China
Lemon    150   Jan  1986   America
Pear     240   Mar  1990   Janpan
avocado  120   Feb  2008   china
```
   
我们来瞧一瞧下面这些简单到爆炸的例子，这个表示打印输出文件的每一整行的内容

```
$ awk '{print $0}' fruit.txt
peach    100   Mar  1997   China
Lemon    150   Jan  1986   America
Pear     240   Mar  1990   Janpan
avocado  120   Feb  2008   china
```

下面这个表示打印输出文件的每一行的第 1 列内容

```
$ awk '{print $1}' fruit.txt
peach
Lemon
Pear
avocado
```
   
下面面这个表示打印输出文件的每一行的第 1 列、第 2 列和第 3 列内容

```
$ awk '{print $1, $2, $3}' fruit.txt
peach 100 Mar
Lemon 150 Jan
Pear 240 Mar
avocado 120 Feb
```
   
其中加入的逗号表示插入输出分隔符，也就是默认的空格
   
**文件的每一行的每一列的内容除了可以用 print 命令打印输出以外，还可以对其进行赋值**
   
```
$ awk '{$2 = "***"; print $0}' fruit.txt
peach *** Mar 1997 China
Lemon *** Jan 1986 America
Pear *** Mar 1990 Janpan
avocado *** Feb 2008 china
```
   
上面的例子就是表示通过对 `$2` 变量进行重新赋值，来隐藏每一行的第 2 列内容，并且用星号 `*` 来代替其输出
   
**在参数列表中加入一些字符串或者转义字符之类的东东**

```
$ awk '{print $1 "\t" $2 "\t" $3}' fruit.txt
peach   100     Mar
Lemon   150     Jan
Pear    240     Mar
avocado 120     Feb
```
   
像上面这样，你可以在 `print`的参数列表中加入一些字符串或者转义字符之类的东东，让输出的内容格式更漂亮，但一定要记住要使用双引号。
   
**awk 内置 NR 变量表示每一行的行号**

```
$ awk '{print NR "\t" $0}' fruit.txt
1   peach    100   Mar  1997   China
2   Lemon    150   Jan  1986   America
3   Pear     240   Mar  1990   Janpan
4   avocado  120   Feb  2008   china
```
   
**awk 内置 NF 变量表示每一行的列数**

```
$ awk '{print NF "\t" $0}' fruit.txt
5   peach    100   Mar  1997   China
5   Lemon    150   Jan  1986   America
5   Pear     240   Mar  1990   Janpan
5   avocado  120   Feb  2008   china
```

**awk 中 $NF 变量的使用**

```
$ awk '{print $NF}' fruit.txt
China
America
Janpan
china
```
   
上面这个 `$NF` 就表示每一行的最后一列，因为 NF 表示一行的总列数，在这个文件里表示有 5 列，然后在其前面加上 `$` 符号，就变成了 `$5` ，表示第 5 列
   
```
$ awk '{print $(NF - 1)}' fruit.txt
1997
1986
1990
2008
```

上面 `$(NF-1)` 表示倒数第 2 列， `$(NF-2)` 表示倒数第 3 列，依次类推。

**现在除了刚才说的有一个 fruit.txt 文件之外，我们又多了一个新文件叫 company.txt 内容如下**

```
yahoo   100 4500
google  150 7500
apple   180 8000
twitter 120 5000
```
   
**我们用 fruit.txt 和 company.txt 两个文件来向你演示 awk 同时处理多个文件的时候有什么效果**

```
$ awk '{print FILENAME "\t" $0}' fruit.txt company.txt
fruit.txt       peach    100   Mar  1997   China
fruit.txt       Lemon    150   Jan  1986   America
fruit.txt       Pear     240   Mar  1990   Janpan
fruit.txt       avocado  120   Feb  2008   china
company.txt     yahoo   100 4500
company.txt     google  150 7500
company.txt     apple   180 8000
company.txt     twitter 120 5000
```
   
当你使用 awk 同时处理多个文件的时候，它会将多个文件合并处理，变量 `FILENAME` 就表示当前文本行所在的文件名称。

看到这里是不是感觉 awk 命令的使用方法真的是简单到爆炸，现在不要太高兴，请举起你的双手跟我一起摇摆。。。哦，不对！请拿起你的双手在电脑上试一试上面这些例子。
你会知道我没有骗你，因为讲了这么多，傻子都会了。。。—_—
   
**BEGIN 关键字的使用**
   
在脚本代码段前面使用 BEGIN 关键字时，它会在开始读取一个文件之前，运行一次 BEGIN 关键字后面的脚本代码段，
BEGIN 后面的脚本代码段只会执行一次，执行完之后 awk 程序就会退出
   
```
$ awk 'BEGIN {print "Start read file"}' /etc/passwd
Start read file
```
   
awk 脚本中可以用多个花括号来执行多个脚本代码，就像下面这样

```
$ awk 'BEGIN {print "Start read file"} {print $0}' /etc/passwd
Start read file
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
```
   
**END 关键字使用方法**

awk 的 END 指令和 BEGIN 恰好相反，在 awk 读取并且处理完文件的所有内容行之后，才会执行 END 后面的脚本代码段

```
$ awk 'END {print "End file"}' /etc/passwd
End file
```
   
一定要多动手在电脑上敲一敲这些命令，对身体好。脑子是个好东西，要多用。。。

```
$ awk 'BEGIN {print "Start read file"} {print $0} END {print "End file"}' /etc/passwd
Start read file
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
End file
```
   
**在 awk 中使用变量**
   
可以在 awk 脚本中声明和使用变量

```
$ awk '{msg="hello world"; print msg}' /etc/passwd
hello world
hello world
hello world
hello world
hello world
```
   
awk 声明的变量可以在任何多个花括号脚本中使用

```
$ awk 'BEGIN {msg="hello world"} {print msg}' /etc/passwd
hello world
hello world
hello world
hello world
hello world
```
   
**在 awk 中使用数学运算**

在 awk 中，像其他编程语言一样，它也支持一些基本的数学运算操作
   
```
$ awk '{a = 12; b = 24; print a + b}' company.txt
36
36
36
36
```

上面这段脚本表示，先声明两个变量 a = 12 和 b = 24，然后用 print 打印出 a 加上 b 的结果。

看到上面的输出结果，你很可能又是一脸的懵逼，为什么会重复输出 4 次同样的计算结果。所以说小时不学好，长大做IT。
知识这东西真到了要用的时候，能亮瞎别人的双眼，好了，不废话。请记住 awk 是针对文件的每一行来执行一次单引号
里面的脚本代码，每读取到一行就会执行一次，文件里面有多少行就会执行多少次，但 BEGIN 和 END 关键字后面的
脚本代码除外，如果被处理的文件中什么都没有，那 awk 就一次都不会执行。。。

**awk 还支持其他的数学运算符**

`+`  加法运算符
`-`  减法运算符
`*`  乘法运算符
`/`  除法运算符
`%`  取余运算符

**在 awk 中使用条件判断**

比如有一个文件 company.txt 内容如下

```
yahoo   100 4500
google  150 7500
apple   180 8000
twitter 120 5000
```

**我们要判断文件的第 3 列数据，也就是平均工资小于 5500 的公司，然后将其打印输出**

```
$ awk '$3 < 5500 {print $0}' company.txt
yahoo   100 4500
twitter 120 5000
```

上面的命令结果就是平均工资小于 5500 的公司名单， `$3 < 5500` 表示当第 3 列字段的内容小于 5500 的时候才会执行后面的 `{print $0}` 代码块

```
$ awk '$1 == "yahoo" {print $0}' company.txt
yahoo   100 4500
```

**awk 还有一些其他的条件操作符如下**

`<`    小于
`<=`   小于或等于
`==`   等于
`!=`   不等于
`>`    大于
`>=`   大于或等于
`~`    匹配正则表达式
`!~`   不匹配正则表达式

**使用 if 指令判断来实现上面同样的效果**

```
$ awk '{if ($3 < 5500) print $0}' company.txt
yahoo   100 4500
twitter 120 5000
```

上面表示如果第 3 列字段小于 5500 的时候就会执行后面的 `print $0`，很像 C 语言和 PHP 的语法对不对。
想到这里有一句话不知当讲不当讲，那就是 PHP 是世界上最好的语言。。。 我可能喝多了，
但是突然想起来我好像从来不喝酒。。。—_—

**在 awk 中使用正则表达式**

在 awk 中支持正则表达式的使用，如果你还对正则表达式不是很了解，请先停下来，上 google 去搜一下。

比如现在我们有这么一个文件 poetry.txt 里面都是我写的诗，不要问我为什么那么的有才华。内容如下：

```
This above all: to thine self be true
There is nothing either good or bad, but thinking makes it so
There’s a special providence in the fall of a sparrow
No matter how dark long, may eventually in the day arrival
```

**使用正则表达式匹配字符串 "There" ，将包含这个字符串的行打印并输出**

```
$ awk '/There/{print $0}' poetry.txt
There is nothing either good or bad, but thinking makes it so
There’s a special providence in the fall of a sparrow
```

**使用正则表达式配一个包含字母 t 和字母 e ，并且 t 和 e 中间只能有任意单个字符的行**

```
$ awk '/t.e/{print $0}' poetry.txt
There is nothing either good or bad, but thinking makes it so
There’s a special providence in the fall of a sparrow
No matter how dark long, may eventually in the day arrival
```

如果只想匹配单纯的字符串 "t.e"， 那正则表达式就是这样的 `/t\.e/` ，用反斜杠来转义 `.` 符号
因为 `.` 在正则表达式里面表示任意单个字符。

**使用正则表达式来匹配所有以 "The" 字符串开头的行**

```
$ awk '/^The/{print $0}' poetry.txt
There is nothing either good or bad, but thinking makes it so
There’s a special providence in the fall of a sparrow
```

在正则表达式中 `^` 表示以某某字符或者字符串开头。

**使用正则表达式来匹配所有以 "true" 字符串结尾的行**

```
$ awk '/true$/{print $0}' poetry.txt
This above all: to thine self be true
```

在正则表达式中 `$` 表示以某某字符或者字符串结尾。

**又一个正则表达式例子如下**

```
$ awk '/m[a]t/{print $0}' poetry.txt
No matter how dark long, may eventually in the day arrival
```

上面这个正则表达式 `/m[a]t/` 表示匹配包含字符 m ，然后接着后面还要包含中间方括号中表示的单个字符 a ，最后还要包含字符 t 的行，输出结果中只有单词 "matter" 符合这个正则表达式的匹配。因为正则表达式 `[a]` 方括号中表示匹配里面的任意单个字符。

**继续上面的一个新例子如下**

```
$ awk '/^Th[ie]/{print $0}' poetry.txt
This above all: to thine self be true
There is nothing either good or bad, but thinking makes it so
There’s a special providence in the fall of a sparrow
```

这个例子中的正则表达式 `/^Th[ie]/`表示匹配以字符串 "Thi" 或者 "The" 开头的行，正则表达式方括号中表示匹配其中的任意单个字符。

**再继续上面的新的用法**

```
$ awk '/s[a-z]/{print $0}' poetry.txt
This above all: to thine self be true
There is nothing either good or bad, but thinking makes it so
There’s a special providence in the fall of a sparrow
```

正则表达式 `/s[a-z]/` 表示匹配包含字符 s 然后后面跟着任意 a 到 z 之间的单个字符的字符串，比如 "se", "so", "sp" 等等。

**正则表达式 `[]` 方括号中还有一些其他用法比如下面这些**

```
[a-zA-Z]  表示匹配小写的 a 到 z 之间的单个字符，或者大写的 A 到 Z 之间的单个字符
[^a-z]    符号 `^` 在方括号里面表示取反，也就是非的意思，表示匹配任何非 a 到 z 之间的单个字符
```

**正则表达式中的星号 `*` 和加号 `+` 的使用方法**

```
$ awk '/go*d/{print $0}' poetry.txt
There is nothing either good or bad, but thinking makes it so
```

上面这个表示匹配包含字符串 "god"，并且中间的字母 "o" 可以出现 0 次或者多次，比如单词 "good" 就符合这个要求。
正则表达式中的 `+` 和星号原理差不多，只是加号表示任意 1 个或者 1 个以上，也就是必须至少要出现一次。

**正则表达式问号 ? 的使用方法**

```
$ awk '/ba?d/{print $0}' poetry.txt
There is nothing either good or bad, but thinking makes it so
```

正则表达式中的问号 ? 表示它前面的字符只能出现 0 次 或者 1 次，也就是可以不出现，也可以出现，但如果有出现也只能出现 1 次。

**正则表达式中的 {} 花括号用法**

```
$ awk '/go{2}d/{print $0}' poetry.txt
There is nothing either good or bad, but thinking makes it so
```

花括号 `{}` 表示规定它前面的字符必须出现的次数，像上面这个 `/go{2}d/` 就表示只匹配字符串 "good"，也就是中间的字母 "o"
必须要出现 2 次。

**正则表达式中的花括号还有一些其他的用法如下**

```
/go{2,3}d/    表示字母 "o" 只能可以出现 2 次或者 3 次
/go{2,10}d/   表示字母 "o" 只能可以出现 2次，3次，4次，5次，6次 ... 一直到 10 次
/go{2,}d/     表示字母 "o" 必须至少出现 2 次或着 2 次以上
```

**正则表达式中的圆括号的用法**

```
$ awk '/th(in){1}king/{print $0}' poetry.txt
There is nothing either good or bad, but thinking makes it so
```

正则表达式中的圆括号表示将多个字符当成一个完整的对象来看待。比如 `/th(in){1}king/` 就表示其中字符串 "in" 必须出现 1 次。而如果不加圆括号就变成了 `/thin{1}king/` 这个就表示其中字符 "n" 必须出现 1 次。

看到这里，如果你对 poetry.txt 件中写的诗比较熟悉，你就会发现。。。我去！这诗根本就不是我写的。所以论多读书是多么的重要。我有幸借用莎士比亚的诗来向你讲解如何在 awk 中使用正则表达式。现在该想想晚上吃什么，晚上去吃火锅。。。—_—

## 使用 awk 的一些总结

因为 awk 算起来也是一种编程语言，它的功能远远不止我们上面讲的这些，awk 还有一些其他比较复杂的功能。但一般我们不建议将 awk 用的太过于复杂。通常面对一些比较复杂的场景我们还是要使用其他的一些工具，比如 shell 脚本，Lua 等等。。。

------------------------------------------------------------------------------------------

By typefo <typefo@qq.com> Update: 2017-04-29 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
