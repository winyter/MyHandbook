# sed 命令

- sed 递归替换当前目录下所有文件，将 x 替换为 y

  ```shell
  find . -type f -print0 | xargs -0 sed -i "s/x/y/g"
  ```

