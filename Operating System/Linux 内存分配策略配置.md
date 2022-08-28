# Linux 内存分配策略配置

## vm.overcommit_memory & vm.overcommit_ratio

overcommit_memory: 有三个值可以配置

- 0: 用户申请内存的时候，系统会判断剩余的内存多少，如果不够的话那么就会失败
- 1: 用户申请内存的时候，系统不进行任何检查任务内存足够用，直到使用内存超过可用内存
- 2: 用户一次申请的内存大小不允许超过可用内存的大小

overcommit_ratio: 仅当 overcommit_memory 为 2 时，该参数才会生效，这个参数决定了系统可用内存的大小

计算公式:

```
(Physical-RAM-Size）* ratio / 100 + (Swap-Size)
  Physical-RAM-Size: 物理内存大小，具体为 free 命令回显中 Mem 行 Total 列的值
  Swap-Size: swap 内存大小，具体为 free 命令回显中 Swap 行 Total 列的值
  ratio: 即 overcommit_ratio 参数的配置值
```

