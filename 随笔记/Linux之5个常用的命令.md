# Linux之5个常用的命令

## 整机性能

- top

  ```sh
  top - 16:19:17 up 19 days, 20:05,  1 user,  load average: 8.85, 8.95, 8.15
  Tasks: 111 total,   2 running, 106 sleeping,   0 stopped,   3 zombie
  %Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
  KiB Mem :  1882012 total,    75108 free,  1740032 used,    66872 buff/cache
  KiB Swap:  4194300 total,  2097580 free,  2096720 used.    30432 avail Mem 
  
    PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                                                                                                     
  25309 root      20   0  247920 126792      0 S  3.7  6.7  82:58.47 containerd-shim                                                                                                                                                                                           
     30 root      20   0       0      0      0 S  3.0  0.0 393:01.02 kswapd0                                                                                                                                                                                                   
    261 root       0 -20       0      0      0 S  1.3  0.0  73:18.32 kworker/0:1H                                                                                                                                                                                              
      6 root      20   0       0      0      0 S  0.3  0.0   2:55.86 ksoftirqd/0                                                                                                                                                                                               
      9 root      20   0       0      0      0 R  0.3  0.0  11:07.74 rcu_sched  
  ```

  - 处理器 cpu 
  - 内存 mem
  - 空闲率 id = idle
  - 系统平均负载率 load average  (a+b+c)/3 *100% 60  重 80 即将宕机

- uptime 

    ```sh
    [root@VM-8-2-centos ~]# uptime
     16:24:02 up 19 days, 20:10,  1 user,  load average: 5.14, 7.53, 7.82
    ```

## 内存

free

```sh
##一般用
[root@VM-8-2-centos ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1837        1539         138           0         159         145
Swap:          4095        2044        2051
```

## 磁盘

df

```sh
#常用
[root@VM-8-2-centos ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        908M     0  908M   0% /dev
tmpfs           919M   24K  919M   1% /dev/shm
tmpfs           919M   11M  909M   2% /run
tmpfs           919M     0  919M   0% /sys/fs/cgroup
/dev/vda1        40G   20G   19G  51% /
tmpfs           184M     0  184M   0% /run/user/0
overlay          40G   20G   19G  51% /var/lib/docker/overlay2/5d705ced68badf1a115eb4a45efa4dd03fe9c169f71e813e33cf3d2bc399081b/merged
overlay          40G   20G   19G  51% /var/lib/docker/overlay2/5e025b96a06c6ff8b3e09aadcf3420cf7e6fb4781d830efb7a7b0a79e6250a75/merged
overlay          40G   20G   19G  51% /var/lib/docker/overlay2/dc5efa564844e9bbe8ea05e2d8fd80c364e706b771b26aa12ca57613ca6d6763/merged
shm              64M     0   64M   0% /var/lib/docker/containers/8a9f8de5dbeda685d2ddd90b27eb7a53e3b639a736089dca901e08be6b504526/mounts/shm
```

## cpu

vmstat -n 2 3  包含但不限于

```sh
[root@VM-8-2-centos ~]# vmstat -n 2 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 4  2 2096892  71956    432  84004 1223 1148 11476  1208   14   25  4  3 77 16  0
 0  5 2096648  74340    268  82840 10544 8166 64594  8438 8482 23035  4 13  0 83  0
 0  1 2093408  78680    280  58976 9868  428 57984   528 8147 21938  2  9  0 89  0
```

> procs r b 进程 数
>
> us  用户 sy 系统 id  空闲

## 磁盘IO

 iostat -xdk 2 3

> 未找到命令可用 yum install sysstat

## 权限

chomd 