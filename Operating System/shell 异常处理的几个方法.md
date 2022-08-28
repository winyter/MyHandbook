# shell 异常处理的几个方法



1. 判断命令返回值

   ```shell
   #!/bin/sh
   cd /home/xxxx/
   if [ "$?"= "0" ]; then
      rm -rf *
   else
      echo "cannot change directory" 1>&2
      exit 1
   fi 
   ```

2. && 和 ||

   &&: 只有在shell1返回0（即正常）时，shell2才会执行，否则shell2根本就不执行

   ||: 在shell1执行失败时，shell2才会执行，否则shell2是不执行的

   ```shell
   cd /home/xxxx/ && rm -rf *
   cd /home/xxxx || error_exit "Cannot change directory"
   rm -rf *
   
   -------
   function error_exit {
     echo "$1" 1>&2
     exit 1
   }
   ```

3. trap