# datetime 模块

```python
# 字符串转 datetime 对象
datetime.datetime.strptime(data_str, "%Y-%m-%d %H:%M:%S")
# datetime 对象转字符串
datetime.xxx.xxx.strftime("%Y-%m-%d %H:%M:%S")
# 获取当前时间时间戳
datetime.datetime.now().timestamp()
# 如果不需要带毫秒
int(datetime.datetime.now().timestamp())
```

