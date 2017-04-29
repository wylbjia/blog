# sort 命令使用详解

sort 命令会将文件的每一行当作一个处理对象，然后将他们从起始字符到末尾依次进行 ASCII 码进行比较，最后按默认升序排列输出

## sort 排序命令的基本用法

```
$ sort [option] file
$ sort [option] --files0-from=F
```

## sort 选项 OPTION 参数详解

**-b, --ignore-leading-blanks**

忽略每一行前面的所有空白字符

**-d, --dictionary-order**

consider only blanks and alphanumeric characters
           
**-f, --ignore-case**

忽略大小写 fold lower case to upper case characters
           
**-g, --general-numeric-sort**

compare according to general numerical value
           
**-i, --ignore-nonprinting**

忽略不可打印字符 consider only printable characters
           
**-M, --month-sort**

将内容当作月份来排序处理 compare (unknown) < 'JAN' < ... < 'DEC'
           
**-h, --human-numeric-sort**

compare human readable numbers (e.g., 2K 1G)
           
**-n, --numeric-sort**

将内容当作纯数字来排序处理

**-R, --random-sort**

随机乱序排列处理，这个选项用来对某一个文件内容进行乱序排列处理，用来打乱某一个文件的数据行

**--random-source=FILE**

get random bytes from FILE

**-r, --reverse**

反向排序 reverse the result of comparisons

**--sort=WORD**

sort according to WORD: general-numeric -g, human-numeric -h, month -M, numeric -n, random -R, version -V

**-V, --version-sort**

natural sort of (version) numbers within text

**--batch-size=NMERGE**

merge at most NMERGE inputs at once; for more use temp files

**-c, --check, --check=diagnose-first**

检查文件内容是否已经被排序过 check for sorted input; do not sort

**-C, --check=quiet, --check=silent`**

like -c, but do not report first bad line

**--compress-program=PROG**

compress temporaries with PROG; decompress them with PROG -d

**--debug**

annotate the part of the line used to sort, and warn about questionable usage to stderr

**--files0-from=F**

read input from the files specified by NUL-terminated names in file F; If F is - then read names from standard input

**-k, --key=KEYDEF**

这个选项通常和 -t 选项配合使用，表示某一个字段域, 每个字段域的默认排序规则为升序排序，它的基本参数格式如下:

```    
sort -k FStart[.CStart][OPTS],FEnd[.CEnd][OPTS]
```

`FStart` 表示字段域，也就是字段域,表示第几个字段开始
`CStart` 表示字段域的内容从第几个字符开始，如果省略则默认以第一个字符开始
`FEnd`   表示字段域，也就是字段域,表示第几个字段结束,如果省略默认为整行的行尾
`CEnd`   表示字段域的内容从第几个字符为结束,如果省略则以该字段域的最后一个字符为结束
`OPTS`   表示其他可选参数组合 `[bdfgiMhnRrV]`

以逗号为分割有两部分，左边部分 `FStart[.CStart][OPTS]` 表示起始标记位，右边部分 `FEnd[.CEnd][OPTS]` 右边第二部分也就是 `FEnd` ,`CEnd` 和 `OPTS` 被省略，则默认结束位置标记位为整行的最后一个字符，而不是当前字段域的最后一个字符

比如有一个文件 company.txt 内容如下,来讲解一些基本用法和使用示例：以空格为分隔，第一列表示公司名称，第二列表示员工人数，第三列表示平均工资

```    
yahoo   180 5500
google  120 4500
taobao  150 5000
tencent 150 7500
baidu   100 3500
twitter 100 4500
apple   200 8000
```

## sed 参数的基本用法和规则

以空格为分隔符，第1个字段域的第1个字符开始，到整行内容的最后一个字符结束

```
$ sort -t ' ' -k 1 company.txt
```

以空格为分隔符，第1个字段域的第2个字符开始，到整行内容的最后一个字符结束

```
$ sort -t ' ' -k 1.2 company.txt
```

以空格为分隔符，第1个字段域的第1一个字符开始，到第1个字段域的最后一个字符结束

```
$ sort -t ' ' -k 1,1 company.txt
```

表示以空格为分隔符，第1个字段域的第1个字符开始，到第1个字段域的第5个字符结束

```
$ sort -t ' ' -k 1.1,1.5 company.txt
```

表示以空格为分隔符，第1个字段域的第2个字符开始，到第3个字段域的第5个字符结束

```
$ sort -t ' ' -k 1.2,3.5 company.txt
```

示例:以公司的名称首字母,按照升序来进行排序

```
$ sort -t ' ' -k 1 company.txt
apple   200 8000
baidu   100 3500
google  120 4500
taobao  150 5000
tencent 150 7500
twitter 100 4500
yahoo   180 5500
```

参数 -t 表示以空格为分隔符对 company.txt 文件中的每一行的内容进行分割，然后 -k 表示以    
分割后的第一个字段的内容也就是公司名称，来进行排序,默认以字母升序的顺序来进行排序

示例:以公司人数升序进行排序，如果人数相同的再按照工资来进行升序排序

```
$ sort -n -t ' ' -k 2 k 3 company.txt
baidu   100 3500
twitter 100 4500
google  120 4500
taobao  150 5000
tencent 150 7500
yahoo   180 5500
apple   200 8000
```

以空格作为分隔符, -n 参数表示将字段内容当作纯数字，可以设定多个 -k 字段来组合排序，
对所有公司的人数来排序，如果人数相同的，则按平均工资来排序

示例：以公司员工数升序排序，如果人数相同的再按照工资的降序来排序

```
$ sort -n -t ' ' -k 2 -k 3r company.txt
twitter 100 4500
google  100 4500
baidu   100 3500
tencent 150 7500
taobao  150 5000
yahoo   180 5500
apple   200 8000
```

这个和上面一例相同，不同的是在第3个字段域也就是工资那一列加上了一个 r 选项，表示反向降序排列

示例:将公司的人数按照升序排序，如果人数相同的再按照工资的降序来进行排序

```
$ sort -n -t ' ' -k 2 -k 3r company.txt
twitter 100 4500
baidu   100 3500
google  120 4500
tencent 150 7500
taobao  150 5000
yahoo   180 5500
apple   200 8000
```

第一个 -k 参数表示按照人数默认的升序进行排序，如果有人数相同的再按照第二个 -k 参数来对工资的降序来进行排序
第二个 -k 参数后面加了一个 r 表示反向排序，因为 sort 默认是升序排序，参数 r 就表示这个字段用降序排序

```
$ sort -t ' ' -k 2n -k 3rn company.txt
twitter 100 4500
baidu   100 3500
google  120 4500
tencent 150 7500
taobao  150 5000
yahoo   180 5500
apple   200 8000
```

-k 参数后面也可以加入 n 参数，表示将这个字段的内容当作纯数字来处理

示例：带有 OPTS 可选参数的一些用法

```
$ sort -t ' ' -k 2n company.txt
```

表示以空格为分隔符，以第2个字段域的第一个字符开始，到整行的最后一个字符结束，并且将该字段域当作纯数字来进行排序处理

```
$ sort -t ' ' -k 2nr company.txt
```

表示以空格为分隔符，以第2个字段域的第一个字符开始，到整行的最后一个字符结束
并且将该字段域当作纯数字然后反向顺序来进行排序处理

```
$ sort -t ' ' -k 2nr -k 3n company.txt
```

表示以空格为分隔符，以第2个字段域的第一个字符开始，到整行的最后一个字符结束,
并且将该字段域当作纯数字然后反向顺序来进行排序处理,如果员工数相同的，再按照
从第3个字段域的第一个字符开始到

**-m, --merge**

merge already sorted files; do not sort

**-o, --output=FILE**

将输出写入到一个文件中 write result to FILE instead of standard output

**-s, --stable**

stabilize sort by disabling last-resort comparison

**-S, --buffer-size=SIZE**

use SIZE for main memory buffer

**-t, --field-separator=SEP**

内容字段分隔符

**-T, --temporary-directory=DIR**

use DIR for temporaries, not $TMPDIR or /tmp; multiple options specify multiple directories

**--parallel=N**

change the number of sorts run concurrently to N

**-u, --unique**

过滤合并相同内容的行，只显示一行，将某一个字段域有相同内容的行进行合并，只保留一行，然后删除其他行，默认比较整行内容是否相同，如果和 -k 参数一起使用，则按照 -k 参数指定的字段域进行比较是否相同，如果有多个 -k 参数字段域，则所有字段域相同才会进行合并

```    
$ cat company.txt
yahoo   180 5500
google  100 4500
google  100 4500
taobao  150 5000
tencent 150 7500
baidu   100 3500
twitter 100 4500
apple   200 8000
```

过滤公司名称相同的行

```
$ sort -t ' ' -k 1,1 -u company.txt
apple   200 8000
baidu   100 3500
google  100 4500
taobao  150 5000
tencent 150 7500
twitter 100 4500
yahoo   180 5500
```

将 company.txt 文件中两个 google 名字相同的公司合并成了一行，此时的 -u 参数过滤重复行是根据 -k 参数指定的字段与来比较的

**-z, --zero-terminated**

line delimiter is NUL, not newline

**--help**

display this help and exit

**--version**

----------------------------------------------------------------------------------------

By typefo typefo@qq.com Update: 2017-04-29 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
