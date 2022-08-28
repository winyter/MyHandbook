# Shell脚本实现进程的自动拉起

> 使用脚本检查某个进程是否在运行，若否，则启动该进程。
>
> 主要思路：根据进程名进行查找（当然如果有两个名字一样的进程就不行了），用 ps + grep 检查进程是否已经存在，同时要注意用grep -v过滤掉当前脚本的进程（因为该进程的路径会作为参数传给这个脚本），还有要过滤掉grep命令产生的子进程（在shell中执行命令时会调用fork产生一个子进程，然后用exec更换进程的映象）
>
> 注意重启进程时直接启动了一个可执行文件：$process_path &
>
> 如果要启动的是shell脚本的话，可以使用：sh $process_path &
>
> 启动完进程之后可以再检查一下进程是否已经成功启动了。
>
> 最后，如果再加上crontab，就可以让脚本定时执行，一旦发现进程没有运行，则启动该进程，这样就实现了进程的自动拉起。

```shell
*/1 * * * * /home/xxx/proc_monitor.sh  /home/xxx/demo &
```

```shell
#!/bin/bash
# name:proc_monitor.sh
# 作用：监控程序，若程序被kill，则自动拉起程序

# 脚本的名字，用以在查找进程时过滤掉当前脚本的进程
script_name="proc_monitor.sh"

# 检查文件是否存在
# @param $1:要监控的程序的目录
function check_file()
{
    if [ $# -ne 1 ]; then
        echo "参数错误，正确使用方法： check_file /dir/file"
        return 1;
    fi

    if [ ! -f $1 ];then
        echo "文件 $1 不存在"
        return 1
    fi
}


# return 0:进程没有运行 1:进程已运行
# @param $1:要监控的程序的目录
function monitor_process()
{
    if [ $# -ne 1 ]; then
        echo "参数错误，正确使用方法：monitor_process /dir/file"
        return 1
    fi

    process_exists=$(ps -ef | grep $1 | grep -v grep | grep -v $script_name | wc -l)
    
    if [ $process_exists -eq 0 ]; then
        return 0
    else
        return 1
    fi
}


# 启动监控程序
# @param $1:要监控的程序的目录function start_monitor()
{
    if [ $# -ne 1 ]; then
        echo "参数错误，正确使用方法：start_monitor /dir/file"
        return 1
    fi

    process_path=$1
    process_name=$(echo $process_path | awk -F / '{print $NF}')
    
    monitor_process $process_name
    if [ $? -eq 0 ]; then
        echo "该进程没有运行：$process_path"
        echo "现将启动进程"

        $process_path &
        #sh $process_path &
        
        monitor_process $process_name
        if [ $? -eq 1 ]; then
            echo "重启进程成功"
        else
            echo "重启进程失败"
        fi
     else
        echo "进程已经启动"
     fi   
}


# 主程序
# @param $1:要监控的程序的目录
if [ $# -ne 1 ]; then
    echo "参数错误，正确使用方法：ecdata_proc_monitor /dir/file"
    exit 1
fi

check_file $1
if [ $? -ne 0 ]; then
    exit 1
fi

start_monitor $1
```

