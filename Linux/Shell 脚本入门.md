　　可以将Shell 终端解释器当作人与计算机硬件之间的`翻译官`，它作为用户与Linux 系统内部的通信媒介，除了能够支持各种变量与参数外，还提供了诸如循环、分支等高级编程语言才有的控制结构特性。要想正确使用Shell 中的这些功能特性，准确下达命令尤为重要。Shell 脚本命令的工作方式有两种：交互式和批处理。

　　**交互式（`Interactive`）：用户每输入一条命令就立即执行。**
　　**批处理（`Batch`）：由用户事先编写好一个完整的Shell 脚本，Shell 会一次性执行脚本中诸多的命令。**

　　在Shell 脚本中不仅会用到前面学习过的很多Linux 命令以及正则表达式、管道符、数据流重定向等语法规则，还需要把内部功能模块化后通过逻辑语句进行处理，最终形成日常所见的Shell 脚本。

　　查看SHELL 变量可以发现当前系统已经默认使用Bash 作为命令行终端解释器了：

```
[root@lynchj tmp]# echo $SHELL
/bin/bash
```

---

### 编写简单的脚本

　　估计读者在看完上文中有关Shell 脚本的复杂描述后，会累觉不爱吧。但是，上文指的是一个高级Shell 脚本的编写原则，其实使用Vim 编辑器把Linux 命令按照顺序依次写入到一个文件中，这就是一个简单的脚本了。

　　例如，如果想查看当前所在工作路径并列出当前目录下所有的文件及属性信息，实现这个功能的脚本应该类似于下面这样：

```
[root@lynchj ~]# vim example.sh
#!/bin/bash
#我的简单脚本
pwd
ls -al
```

　　`Shell 脚本`文件的名称可以任意，但为了避免被误以为是普通文件，建议将`.sh 后缀`加上，以表示是一个脚本文件。在上面的这个`example.sh 脚本`中实际上出现了三种不同的元素：第一行的脚本声明（`#!`）用来告诉系统使用哪种Shell 解释器来执行该脚本；第二行的注释信息（`#`）是对脚本功能和某些命令的介绍信息，使得自己或他人在日后看到这个脚本内容时，可以快速知道该脚本的作用或一些警告信息；第三、四行的可执行语句也就是我们平时执行的Linux 命令了。什么？！你们不相信这么简单就编写出来了一个脚本程序，那我们来执行一下看看结果：

```
[root@lynchj ~]# bash base.sh
/usr/local/tmp
total 16
drwxr-xr-x.  2 root root   51 May 16 13:53 .
drwxr-xr-x. 13 root root 4096 May 14 11:24 ..
-rwxr--r--.  1 root root   38 May 16 13:39 base.sh
-rwxr--r--.  1 root root  134 May 16 13:53 example.sh
-rw-r--r--.  1 root root   54 May 16 13:37 tmp.txt
```

　　除了上面用`bash` 解释器命令直接运行Shell 脚本文件外，第二种运行脚本程序的方法是通过输入完整路径的方式来执行。但默认会因为权限不足而提示报错信息，此时只需要为脚本文件增加执行权限即可（`chmod u+x 脚本名称`）：

```
[root@lynchj tmp]# ./base.sh
-bash: ./base.sh: Permission denied
[root@lynchj tmp]# chmod u+x base.sh
[root@lynchj tmp]# ./base.sh
/usr/local/tmp
total 16
drwxr-xr-x.  2 root root   51 May 16 13:53 .
drwxr-xr-x. 13 root root 4096 May 14 11:24 ..
-r-x------.  1 root root   38 May 16 13:39 base.sh
-rwxr--r--.  1 root root  134 May 16 13:53 example.sh
-rw-r--r--.  1 root root   54 May 16 13:37 tmp.txt
```

---

### 接收用户的参数

　　但是，像上面这样的脚本程序只能执行一些预先定义好的功能，未免太过死板了。为了让Shell 脚本程序更好地满足用户的一些实时需求，以便灵活完成工作，必须要让脚本程序能够像之前执行命令时那样，接收用户输入的参数。

　　其实，Linux 系统中的Shell 脚本语言早就考虑到了这些，已经内设了用于接收参数的变量，变量之间可以使用空格间隔。例如`$0` 对应的是当前Shell 脚本程序的名称，`$#`对应的是总共有几个参数，`$*`对应的是所有位置的参数值，`$?`对应的是显示上一次命令的执行返回值，而`$1、$2、$3……`则分别对应着第N 个位置的参数值，如下图所示：

![Shell 脚本程序中的参数位置变量](http://img.lynchj.com/f12d511093114245b02cfa48efc95b06.png)

　　理论过后我们来练习一下。尝试编写一个脚本程序示例，通过引用上面的变量参数来看下真实效果：

```
[root@lynchj tmp]# vim example.sh
#!/bin/bash
echo "当前脚本名称为$0"
echo "总共有$#个参数，分别是$*。"
echo "第1 个参数为$1，第5 个为$5。"
[root@lynchj tmp]# sh example.sh one two haha 呵呵 哦哦 hello
当前脚本名称为example.sh
总共有6个参数，分别是one two haha 呵呵 哦哦 hello。
第1 个参数为one，第5 个为哦哦。
```

---

### 判断用户的参数

　　学习是一个登堂入室、由浅入深的过程。在学习完Linux 命令、掌握Shell 脚本语法变量和接收用户输入的信息之后，就要踏上新的高度—能够进一步处理接收到的用户参数。

　　系统在执行`mkdir 命令`时会判断用户输入的信息，即判断用户指定的文件夹名称是否已经存在，如果存在则提示报错；反之则自动创建。`Shell 脚本`中的`条件测试语法`可以判断表达式是否成立，若条件成立则返回数字`0`，否则便返回`其他随机数值`。条件测试语法的执行格式如下图所示。切记，条件表达式两边均应有一个空格。

![条件测试语句的执行格式](http://img.lynchj.com/184b6a32fff3445fa1d2d49d23038976.png)

　　按照测试对象来划分，条件测试语句可以分为4 种：
　　➢ **文件测试语句；**
　　➢ **逻辑测试语句；**
　　➢ **整数值比较语句；**
　　➢ **字符串比较语句；**

* 文件测试即使用指定条件来判断文件是否存在或权限是否满足等情况的运算符，具体的参数如下表所示：

| 运算符 | 作用 |
| --- | --- |
| -d | 测试文件是否为目录类型 |
| -e | 测试文件是否存在 |
| -f | 判断是否为一般文件 |
| -r | 测试当前用户是否有权限读取 |
| -w | 测试当前用户是否有权限写入 |
| -x | 测试当前用户是否有权限执行 |

　　下面使用文件测试语句来判断`/usr/local/tmp` 是否为一个目录类型的文件，然后通过`Shell 解释器`的内设`$?`变量显示上一条命令执行后的返回值。如果返回值为`0`，则目录存在；如果返回值为`非零的值`，则意味着目录不存在：

```
[root@lynchj tmp]# [ -d /usr/local/tmp ]
[root@lynchj tmp]# echo $?
0
```

　　再使用文件测试语句来判断`/usr/local/tmp` 是否为一般文件，如果返回值为`0`，则代表文件存在，且为一般文件：

```
[root@lynchj tmp]# [ -f /usr/local/tmp ]
[root@lynchj tmp]# echo $?
1
```

* 逻辑语句用于对测试结果进行逻辑分析，根据测试结果可实现不同的效果。例如在Shell终端中逻辑`与`的运算符号是`&&`，它表示当前面的命令执行成功后才会执行它后面的命令，因此可以用来判断`/usr/local/tmp` 文件是否存在，若存在则输出Exist 字样。

```
[root@lynchj tmp]# [ -e /usr/local/tmp/ ] && echo "Exist"
Exist
```

　　除了逻辑`与`外，还有逻辑`或`，它在Linux 系统中的运算符号为`||`，表示当前面的命令执行失败后才会执行它后面的命令，因此可以用来结合系统环境变量`USER` 来判断当前登录的用户是否为非管理员身份：

```
[root@lynchj tmp]# echo $USER
root
[root@lynchj tmp]# [ $USER = root ] || echo "user"
[root@lynchj tmp]# su - guoqingsong
Last login: Wed May 16 10:58:08 CST 2018 on pts/0
[guoqingsong@lynchj ~]$ [ $USER = root ] || echo "user"
user
```

　　第三种逻辑语句是`非`，在Linux 系统中的运算符号是一个叹号（`!`），它表示把条件测试中的判断结果取相反值。也就是说，如果原本测试的结果是正确的，则将其变成错误的；原本测试错误的结果则将其变成正确的。

我们现在切换到一个普通用户的身份，再判断当前用户是否为一个非管理员的用户。由于判断结果因为两次否定而变成正确，因此会正常地输出预设信息：

```
[guoqingsong@lynchj ~]$ exit
logout
[root@lynchj tmp]# [ ! $USER = root ] || echo "administrator"
administrator
```

* 整数比较运算符仅是对数字的操作，不能将数字与字符串、文件等内容一起操作，而且不能想当然地使用日常生活中的等号、大于号、小于号等来判断。因为等号与赋值命令符冲突，大于号和小于号分别与输出重定向命令符和输入重定向命令符冲突。因此一定要使用规范的整数比较运算符来进行操作。可用的整数比较运算符如下表所示。

| 运算符 | 作用 |
| --- | --- |
| -eq | 是否等于 |
| -ne | 是否不等于 |
| -gt | 是否大于 |
| -ge | 是否大于或等于 |
| -lt | 是否小于 |
| -le | 是否等于或小于 |

　　接下来小试牛刀。我们先测试一下10 是否大于10 以及10 是否等于10（通过输出的返回值内容来判断）：

```
[root@lynchj tmp]# [ 10 -gt 10 ]
[root@lynchj tmp]# echo $?
1
[root@lynchj tmp]# [ 10 -ge 10 ]
[root@lynchj tmp]# echo $?
0
```

　　我们使用`free 命令`，它可以用来获取当前系统正在使用及可用的内存量信息。接下来先使用`free -m 命令`查看内存使用量情况（单位为MB），然后通过`grep Mem:`命令过滤出剩余内存量的行，再用`awk '{print $4}'`命令只保留第四列，最后用`FreeMem=`语句``的方式把语句内执行的结果赋值给变量。

```
[root@lynchj tmp]# free -m
             total       used       free     shared    buffers     cached
Mem:          1987       1790        197         14          9        716
-/+ buffers/cache:       1064        923
Swap:         2047          0       2047
[root@lynchj tmp]# free -m | grep Mem:
Mem:          1987       1790        197         14          9        716
[root@lynchj tmp]# free -m | grep Mem: | awk '{print $4}'
197
[root@lynchj tmp]# FreeMen=`free -m | grep Mem: | awk '{print $4}'`
[root@lynchj tmp]# echo $FreeMen
196
[root@lynchj tmp]# [ $FreeMen -lt 1024 ] && echo "内存快炸了"
内存快炸了
```

* 字符串比较语句用于判断测试字符串是否为空值，或两个字符串是否相同。它经常用来判断某个变量是否未被定义（即内容为空值），理解起来也比较简单。字符串比较中常见的运算符如下表所示：

| 运算符 | 作用 |
| --- | --- |
| = | 比较字符串内容是否相同 |
| != | 比较字符串内容是否不同 |
| -z | 判断字符串内容是否为空 |

---

### 流程控制语句

　　尽管此时可以通过使用Linux 命令、管道符、重定向以及条件测试语句来编写最基本的Shell 脚本，但是这种脚本并不适用于生产环境。原因是它不能根据真实的工作需求来调整具体的执行命令，也不能根据某些条件实现自动循环执行。例如，我们需要批量创建1000位用户，首先要判断这些用户是否已经存在；若不存在，则通过循环语句让脚本自动且依次创建他们。

> 　　接下来我们通过if、for、while、case 这4 种流程控制语句来学习编写难度更大、功能更强的Shell 脚本。

##### if 条件测试语句

　　`if 条件`测试语句可以让脚本根据实际情况自动执行相应的命令。从技术角度来讲，if 语句分为单分支结构、双分支结构、多分支结构；其复杂度随着灵活度一起逐级上升。

　　if 条件语句的单分支结构由`if`、`then`、`fi` 关键词组成，而且只在条件成立后才执行预设的命令，相当于口语的`如果……那么……`。单分支的if 语句属于最简单的一种条件判断结构，语法格式如下图所示：![单分支的if 语句](http://img.lynchj.com/4bf863f289a840a28dfef5aa00f8b070.png)

　　下面使用单分支的if 条件语句来判断`/usr/local/tmp` 文件是否存在，若存在就结束条件判断和整个Shell 脚本，反之则去创建这个目录：

```
[root@lynchj tmp]# vim example.sh
#!/bin/bash
DIR="/usr/local/tmp"
if [ ! -e $DIR ]
then
mkdir -p $DIR
fi
[root@lynchj tmp]# ./example.sh
```

　　`if 条件语句`的双分支结构由`if`、`then`、`else`、`fi` 关键词组成，它进行一次条件匹配判断，如果与条件匹配，则去执行相应的预设命令；反之则去执行不匹配时的预设命令，相当于口语的`如果……那么……或者……那么……`。if 条件语句的双分支结构也是一种很简单的判断结构，语法格式如下图所示：

![双分支的if 语句](http://img.lynchj.com/5e42c1dfef974b78aa64ea195ebf1b15.png)

```
[root@lynchj tmp]# vim example.sh
#!/bin/bash
DIR="/usr/local/tmp"
if [ ! -e $DIR ]
then
mkdir -p $DIR
else
echo "已经存在了"
fi
[root@lynchj tmp]# ./example.sh
已经存在了
```

　　`if 条件`语句的多分支结构由`if`、`then`、`else`、`elif`、`fi` 关键词组成，它进行多次条件匹配判断，这多次判断中的任何一项在匹配成功后都会执行相应的预设命令，相当于口语的`如果……那么……如果……那么……`。if 条件语句的多分支结构是工作中最常使用的一种条件判断结构，尽管相对复杂但是更加灵活，语法格式如下图所示：

![多分支的if 语句](http://img.lynchj.com/8063ed3f04bc4b2796d8b2581d6ac400.png)

　　下面使用多分支的if 条件语句来判断用户输入的分数在哪个成绩区间内，然后输出如`Excellent`、`Pass`、`Fail` 等提示信息。在Linux 系统中，`read` 是用来`读取用户输入信息`的命令，能够把接收到的用户输入信息赋值给后面的指定变量，`-p 参数`用于向用户显示一定的提示信息。在下面的脚本示例中，只有当用户输入的分数大于等于85 分且小于等于100 分，才输出Excellent 字样；若分数不满足该条件（即匹配不成功），则继续判断分数是否大于等于70 分且小于等于84 分，如果是，则输出Pass 字样；若两次都落空（即两次的匹配操作都失败了），则输出Fail 字样：

```
[root@lynchj tmp]# vim example.sh
#!/bin/bash
read -p "Enter your score（0-100）：" GRADE
if [ $GRADE -ge 85 ] && [ $GRADE -le 100 ] ; then
echo "$GRADE is Excellent"
elif [ $GRADE -ge 70 ] && [ $GRADE -le 84 ] ; then
echo "$GRADE is Pass"
else
echo "$GRADE is Fail"
fi
[root@lynchj ~]# bash chkscore.sh
Enter your score（0-100）：88
88 is Excellent
[root@lynchj ~]# bash chkscore.sh
Enter your score（0-100）：80
80 is Pass
```

　　下面执行该脚本。当用户输入的分数分别为30 和200 时，其结果如下：

```
[root@lynchj tmp]# ./example.sh
Enter your score（0-100）：30
30 is Fail
[root@lynchj tmp]# ./example.sh
Enter your score（0-100）：200
200 is Fail
```

　　为什么输入的分数为200 时，依然显示Fail 呢？原因很简单—没有成功匹配脚本中的两个条件判断语句，因此自动执行了最终的`兜底策略`。可见，这个脚本还不是很完美，读者可自行完善这个脚本，使得用户在输入大于100 或小于0 的分数时，给予Error 报错字样的提示。

##### for 条件循环语句

　　`for` 循环语句允许脚本一次性读取多个信息，然后逐一对信息进行操作处理，当要处理的数据有范围时，使用for 循环语句再适合不过了。for 循环语句的语法格式如下图所示：

![for 循环语句的语法格式](http://img.lynchj.com/e794bd9429bf4b30b63c14f8b80f1a47.png)

　　下面使用for 循环语句从列表文件中读取多个用户名，然后为其逐一创建用户账户并设置密码。首先创建用户名称的列表文件`users.txt`，每个用户名称单独一行。读者可以自行决定具体的用户名称和个数：

```
[root@lynchj tmp]# vim users.txt
andy
barry
carl
duke
eric
george
[root@lynchj tmp]# vim example.sh
#!/bin/bash
read -p "Enter The Users Password : " PASSWD
for UNAME in `cat users.txt`
do
id $UNAME &> /dev/null
if [ $? -eq 0 ]
then
echo "Already exists"
else
useradd $UNAME &> /dev/null
echo "$PASSWD" | passwd --stdin $UNAME &> /dev/null
if [ $? -eq 0 ]
then
echo "$UNAME , Create success"
else
echo "$UNAME , Create failure"
fi
fi
done
```

　　执行批量创建用户的Shell 脚本example.sh，在输入为账户设定的密码后将由脚本自动检查并创建这些账户。由于已经将多余的信息通过输出重定向符转移到了`/dev/null 黑洞`文件中，因此在正常情况下屏幕窗口除了`用户账户创建成功`（Create success）的提示后不会有其他内容。

　　在Linux 系统中，`/etc/passwd` 是用来保存用户账户信息的文件。如果想确认这个脚本是否成功创建了用户账户，可以打开这个文件，看其中是否有这些新创建的用户信息。

　　还记得在学习双分支`if 条件`语句时，用到的那个测试主机是否在线的脚本，既然我们现在已经掌握了`for 循环`语句，不妨做些更酷的事情，比如尝试让脚本从文本中自动读取主机列表，然后自动逐个测试这些主机是否在线。

```
[root@linuxprobe ~]# vim ipadds.txt
192.168.1.10
192.168.1.11
192.168.1.12
```

　　前面的双分支`if 条件`语句与`for 循环`语句相结合，让脚本从主机列表文件`ipadds.txt`中自动读取IP 地址（用来表示主机）并将其赋值给`HLIST 变量`，从而通过判断`ping 命令`执行后的返回值来逐个测试主机是否在线。脚本中出现的`$（命令）`是一种完全类似于转义字符中反引号**\`命令\`**的Shell 操作符，效果同样是执行括号或双引号括起来的字符串中的命令。大家在编写脚本时，多学习几种类似的新方法，可在工作中大显身手：

```
[root@lynchj tmp]# vim CheckHosts.sh
#!/bin/bash
HLIST=$(cat ~/ipadds.txt)
for IP in $HLIST
do
ping -c 3 -i 0.2 -W 3 $IP &> /dev/null
if [ $? -eq 0 ] ; then
echo "Host $IP is On-line."
else
echo "Host $IP is Off-line."
fi
done
[root@lynchj tmp]# ./CheckHosts.sh
Host 192.168.1.10 is On-line.
Host 192.168.1.11 is Off-line.
Host 192.168.1.12 is Off-line.
```

##### while 条件循环语句

　　`while 条件循环`语句是一种让脚本根据某些条件来重复执行命令的语句，它的循环结构往往在执行前并不确定最终执行的次数，完全不同于`for 循环`语句中有目标、有范围的使用场景。`while 循环`语句通过判断条件测试的真假来决定是否继续执行命令，若条件为真就继续执行，为假就结束循环。`while 语句`的语法格式如下图所示：

![while 循环语句的语法格式](http://img.lynchj.com/1125369c03154f12a596b0857e77ad0c.png)

　　接下来结合使用多分支的`if 条件`测试语句与`while 条件`循环语句，编写一个用来猜测数值大小的脚本`Guess.sh`。该脚本使用`$RANDOM 变量`来调取出一个随机的数值（范围为0～32767），将这个随机数对1000 进行取余操作，并使用`expr 命令`取得其结果，再用这个数值与用户通过`read命令`输入的数值进行比较判断。这个判断语句分为三种情况，分别是判断用户输入的数值是等于、大于还是小于使用`expr 命令`取得的数值。当前，现在这些内容不是重点，我们当前要关注的是`while 条件`循环语句中的条件测试始`终为true`，因此判断语句会无限执行下去，直到用户输入的数值`等于`expr 命令取得的数值后，这两者相等之后才运行`exit 0 命令`，终止脚本的执行。

```
[root@lynchj tmp]# vim Guess.sh
#!/bin/bash
PRICE=$(expr $RANDOM % 1000)
TIMES=0
echo "商品实际价格为0-999 之间，猜猜看是多少？"
while true
do
read -p "请输入您猜测的价格数目：" INT
let TIMES++
if [ $INT -eq $PRICE ] ; then
echo "恭喜您答对了，实际价格是 $PRICE"
echo "您总共猜测了 $TIMES 次"
exit 0
elif [ $INT -gt $PRICE ] ; then
echo "太高了！"
else
echo "太低了！"
fi
done
```

　　在这个Guess.sh 脚本中，我们添加了一些交互式的信息，从而使得用户与系统的互动性得以增强。而且每当循环到`let TIMES++`命令时都会让TIMES 变量内的数值加1，用来统计循环总计执行了多少次。这可以让用户得知总共猜测了多少次之后，才猜对价格。

```
[root@lynchj tmp]# ./Guess.sh
商品实际价格为0-999 之间，猜猜看是多少？
请输入您猜测的价格数目：999
太高了！
请输入您猜测的价格数目：500
太低了！
请输入您猜测的价格数目：750
太高了！
请输入您猜测的价格数目：650
太低了！
请输入您猜测的价格数目：700
太低了！
请输入您猜测的价格数目：725
太高了！
请输入您猜测的价格数目：715
太低了！
请输入您猜测的价格数目：720
太高了！
请输入您猜测的价格数目：718
太低了！
请输入您猜测的价格数目：719
恭喜您答对了，实际价格是 719
您总共猜测了 10 次
```

##### case 条件测试语句

　　如果您之前学习过C 语言，看到这一小节的标题肯定会会心一笑`这不就是switch 语句嘛！`是的，case 条件测试语句和switch 语句的功能非常相似！case 语句是在多个范围内匹配数据，若匹配成功则执行相关命令并结束整个条件测试；而如果数据不在所列出的范围内，则会去执行星号（`*`）中所定义的默认命令。case 语句的语法结构如下图所示：

![case 条件测试语句的语法结构](http://img.lynchj.com/99b933f4e6aa4d65bc183a900d8f73e3.png)

　　在前文介绍的Guess.sh 脚本中有一个致命的弱点—只能接受数字！您可以尝试输入一个字母，会发现脚本立即就崩溃了。原因是字母无法与数字进行大小比较，例如，`a 是否大于等于3`这样的命题是完全错误的。我们必须有一定的措施来判断用户的输入内容，当用户输入的内容不是数字时，脚本能予以提示，从而免于崩溃。

　　通过在脚本中组合使用case 条件测试语句和通配符，完全可以满足这里的需求。接下来我们编写脚本`Checkkeys.sh`，提示用户输入一个字符并将其赋值给变量`KEY`，然后根据变量KEY 的值向用户显示其值是字母、数字还是其他字符。

```
[root@lynchj tmp]# vim Checkkeys.sh
#!/bin/bash
read -p "请输入一个字符，并按Enter 键确认：" KEY
case "$KEY" in
[a-z]|[A-Z])
echo "您输入的是 字母。"
;;
[0-9])
echo "您输入的是 数字。"
;;
*)
echo "您输入的是 空格、功能键或其他控制字符。"
esac
[root@lynchj tmp]# bash Checkkeys.sh
请输入一个字符，并按Enter 键确认：5
您输入的是 数字。
[root@lynchj tmp]# bash Checkkeys.sh
请输入一个字符，并按Enter 键确认：d
您输入的是 字母。
[root@lynchj tmp]# bash Checkkeys.sh
请输入一个字符，并按Enter 键确认：fd^H^H
您输入的是 空格、功能键或其他控制字符。
```

---

> 参考资料：《Linux就该这么学》
