# CPU频率

1. Performance.  不考虑耗电，只用最高频率。
2. Interactive.  直接上最高频率，然后看CPU负荷慢慢降低。
3. Powersave.    通常以最低频率运行，流畅度会受影响，一般不会用这个吧！
4. Userspace.    可以在用户空间手动调节频率。
5. Ondemand.    定期检查负载，根据负载来调节频率。

```
cd  /sys/devices/system/cpu/cpu0/cpufreq/
cat  scaling_available_governors  #支持的cpu调频方式
echo xxx > scaling_governor       #设置cpu动态调频方式
```

# Kernel Log打印等级
```
echo 8 4 1 7 > /proc/sys/kernel/printk
```

# Android 10 init进程打印log

+ 修改BoardConfig.mk  
```
BOARD_KERNEL_CMDLINE += printk.devkmsg=on
```