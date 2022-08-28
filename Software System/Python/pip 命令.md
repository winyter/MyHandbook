# pip 命令

```
pip [download/install]
 -i: 指定下载源
 -d: 指定下载目录
# 导出当前 Python 环境到一个文件
pip freeze > requirements.txt
# 下载 Python 环境的所有包到一个目录
pip download -d C:\Users\Administrator\Desktop\我的脚本\ZTCHS_Checker\pack -r requirements.txt# 导出当前 Python 环境到一个文件
pip freeze > requirements.txt
# 下载 Python 环境的所有包到一个目录
pip download -d C:\Users\Administrator\Desktop\我的脚本\ZTCHS_Checker\pack -r requirements.txt
# 指定清华镜像站，下载包到一个指定目录
pip download -d C:\Users\Administrator\Desktop\我的脚本\ZTCHS_Checker\pack -i https://pypi.tuna.tsinghua.edu.cn/simple setuptools
# 使用清华镜像站更新一个包
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```

