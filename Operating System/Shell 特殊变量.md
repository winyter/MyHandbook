# Shell 特殊变量

## Shell 特殊变量及其含义

| 变量      | 含义                                                         |
| :-------- | :----------------------------------------------------------- |
| \$0        | 当前脚本的文件名。|
| \$n（n≥1） | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是 $1，第二个参数是 $2。 |
| \$#        | 传递给脚本或函数的参数个数。|
| \$*        | 传递给脚本或函数的所有参数。|
| \$@        | 传递给脚本或函数的所有参数。当被双引号" "包含时，\$@ 与 \$* 稍有不同，在下方小节详解 |
| \$?        | 上个命令的退出状态，或函数的返回值，我们将在下方详细讲解。|
| \$\$        | 当前 Shell 进程 ID。对于 Shell 脚本，就是这些脚本所在的进程 ID。  |

演示示例：

1. 给脚本文件传递参数

   编写下面的代码，并保存为 test.sh：

   ```bash
   #!/bin/bash
   echo "Process ID: $$"
   echo "File Name: $0"
   echo "First Parameter : $1"
   echo "Second Parameter : $2"
   echo "All parameters 1: $@"
   echo "All parameters 2: $*"
   echo "Total: $#"
   ```

   运行 test.sh，并附带参数：

   ```bash
   [mozhiyan@localhost demo]$ . ./test.sh Shell Linux
   Process ID: 5943
   File Name: bash
   First Parameter : Shell
   Second Parameter : Linux
   All parameters 1: Shell Linux
   All parameters 2: Shell Linux
   Total: 2
   ```

2. 给函数传递参数

   编写下面的代码，并保存为 test.sh：

   ```shell
   #!/bin/bash
   #定义函数
   function func(){
     echo "Language: $1"
     echo "URL: $2"
     echo "First Parameter : $1"
     echo "Second Parameter : $2"
     echo "All parameters 1: $@"
     echo "All parameters 2: $*"
     echo "Total: $#"
   }
   #调用函数
   func Java http://c.biancheng.net/java/
   ```

   运行结果为：

   ```
   Language: Java
   URL: http://c.biancheng.net/java/
   First Parameter : Java
   Second Parameter : http://c.biancheng.net/java/
   All parameters 1: Java http://c.biancheng.net/java/
   All parameters 2: Java http://c.biancheng.net/java/
   Total: 2
   ```



## \$* 和 \$@ 之间的区别

当 \$* 和 \$@ 不被双引号" "包围时，它们之间没有任何区别，都是将接收到的每个参数看做一份数据，彼此之间以空格来分隔。

但是当它们被双引号" "包含时，就会有区别了：

- "∗"会将所有的参数从整体上看做一份数据，而不是把每个参数都看做一份数据。
- "@"仍然将每个参数都看作一份数据，彼此之间是独立的。

如下示例

如果使用 echo 直接输出"∗ " 和 " *"和"∗"和"@"做对比，是看不出区别的；但如果使用 for 循环来逐个输出数据，立即就能看出区别来。

```shell
#!/bin/bash
 
echo "-- \$* 演示 ---"
for i in "$*"; do
 echo $i
done
 
echo "-- \$@ 演示 ---"
for i in "$@"; do
 echo $i
done
```

执行脚本，输出结果如下所示：

```shell
$ chmod +x test.sh
$ ./test.sh 1 2 3
-- $* 演示 ---
1 2 3
-- $@ 演示 ---
1
2
3
```



## \$? 变量详解

$? 是一个特殊变量，用来获取上一个命令的退出状态，或者上一个函数的返回值。

所谓退出状态，就是上一个命令执行后的返回结果。退出状态是一个数字，一般情况下，大部分命令执行成功会返回 0，失败返回 1，这和C语言的 main() 函数是类似的。

不过，也有一些命令返回其他值，表示不同类型的错误。

1. 获取上一个命令的退出状态

   编写下面的代码，并保存为 test.sh：

   ```shell
   #!/bin/bash
   if [ "$1" == 100 ]
   then
     exit 0 #参数正确，退出状态为0
   else
     exit 1 #参数错误，退出状态1
   fi
   ```

   exit表示退出当前 Shell 进程，我们必须在新进程中运行 test.sh，否则当前 Shell 会话（终端窗口）会被关闭，我们就无法取得它的退出状态了。

   例如，运行 test.sh 时传递参数 100：

   ```shell
   [mozhiyan@localhost ~]$ cd demo
   [mozhiyan@localhost demo]$ bash ./test.sh 100 #作为一个新进程运行
   [mozhiyan@localhost demo]$ echo $?
   0
   ```

   再如，运行 test.sh 时传递参数 89：

   ```shell
   [mozhiyan@localhost demo]$ bash ./test.sh 89 #作为一个新进程运行
   [mozhiyan@localhost demo]$ echo $?
   1
   ```

2. 获取函数返回值

   编写下面的代码，并保存为 test.sh：

   ```shell
   #!/bin/bash
   #得到两个数相加的和
   function add(){
     return `expr $1 + $2`
   }
   add 23 50 #调用函数
   echo $? #获取函数返回值
   
   === 运行结果 ===
   73
   ```

   > 有 C++、C#、Java 等编程经验的读者请注意：严格来说，Shell 函数中的 return 关键字用来表示函数的退出状态，而不是函数的返回值；Shell 不像其它编程语言，没有专门处理返回值的关键字。
