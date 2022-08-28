# 零碎的 Git 知识

## 常用命令

```shell
# 从版本库中删除某个文件
|-- git rm xxx
|-- git commit -m " remove xxx"
```



## .gitignore 文件

- 配置示例

  ```
  # 忽略 .a 文件
  *.a
  
  # 但否定忽略 lib.a, 尽管已经在前面忽略了 .a 文件
  !lib.a
  
  # 仅在当前目录下忽略 TODO 文件， 但不包括子目录下的 subdir/TODO
  /TODO
  
  # 忽略 build/ 文件夹下的所有文件
  build/
  
  # 忽略 doc/notes.txt, 不包括 doc/server/arch.txt
  doc/*.txt
  
  # 忽略所有的 .pdf 文件 在 doc/ directory 下的
  doc/**/*.pdf
  ```

- 配置模板

  ```
  Github 上为开发者提供了各种环境以及各种编程语言的 gitignore 文件配置模板：https://github.com/github/gitignore
  python项目的gitgnore文件 url:https://github.com/github/gitignore/blob/master/Python.gitignore
  ```

- 全局`.gitignore`设置方法( 这个文件将作用于所有 git 项目，并且作用于项目实例中的所有被跟踪的目录)

  ```
  git config --global core.excludesfile ~/.gitignore
  ```

- 局部`.gitignore`直接在项目目录中创建文件即可

