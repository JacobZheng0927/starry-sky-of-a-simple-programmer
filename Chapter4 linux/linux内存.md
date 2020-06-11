top命令

load average 任务数/核数

https://www.cnblogs.com/niuben/p/12017242.html



ps -ef|grep Coa



堆外内存



## 附录

### Linux常用命令

#### 查询核数

```bash
# 查看物理CPU个数
cat /proc/cpuinfo | grep "physical id" | uniq | wc -l 
# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo |grep "processor"|wc -l
# 查看逻辑CPU的个数
cat /proc/cpuinfo | grep "cpu cores" | uniq
```

