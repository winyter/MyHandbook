# iconv 命令详解

- 功能

  对于给定文件，把它的内容从一种编码转换成另一种编码

- 描述

  -f encoding :把字符从encoding编码开始转换。 

  -t encoding :把字符转换到encoding编码。 

  -l :列出已知的编码字符集合 

  -o file :指定输出文件 

  -c :忽略输出的非法字符 

  -s :禁止警告信息，但不是错误信息 

  --verbose :显示进度信息 

  -f和-t所能指定的合法字符在-l选项的命令里面都列出来了。 

- 举例

  列出当前支持的字符编码：

  ```shell
  iconv -l
  ```

  将文件file1转码,转后文件输出到fil2中： 

  ```shell
  iconv  -f EUC-JP-MS -t UTF-8 file1 -o file2   //没-o那么会输出到标准输出.
  ```

  实际需求，从hive中取出的数据是utf8的，要load到mysql中，gbk编码。所以在load之前要先对文件进行转码。

  ```shell
  mysql_cmd = "iconv -c -f utf-8 -t gbk ./data/al_ver_" + yesterday_time + ".xls -o ./data/GBK_al_ver_" + yesterday_time + ".xls "
  print(mysql_cmd)
  os.system(mysql_cmd)
   
  mysql_cmd = "mysql -h60.28.200.78 -uroot -pyeelion -A LogStat_RT  -e \"load data local  infile \'./data/GBK_al_ver_" + yesterday_time + ".xls ' into table HiveData_508\""
  print(mysql_cmd)
  os.system(mysql_cmd)
  ```

