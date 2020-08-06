---
description: >-
  线上的一个日志实时输出的程序曾经出过这样一个问题，刚开始上线java程序占用的`CPU`的资源很少,但是到了整点的时候,CPU直线飙高，直接到达`100%`根本没有要下降的趋势，唯一的方法只能杀掉它了，后面在借助`jstack`与`top`排查到线程然后定位到某行代码出的问题。
---

# 定位导致CPU飙升的问题过程

## 排查过程

使用`jps`找到程序的`pid`

```text
$ jps -l -m | grep logdir2
22169 galaxy-log-online-0.1-SNAPSHOT-all.jar 3002 /logdir2
```

找到`22169`进程ID

找到CPU过高的线程

```text
$ top -H -p 22169
top - 19:03:22 up 156 days,  5:57,  4 users,  load average: 1.00, 2.84, 4.25
Threads:  15 total,   0 running,  15 sleeping,   0 stopped,   0 zombie
%Cpu(s): 99.4 us, 12.6 sy,  0.0 ni, 62.6 id,  4.8 wa,  0.0 hi,  2.6 si,  0.0 st
KiB Mem :  8010456 total,   206760 free,  1079668 used,  6724028 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  6561460 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                       
22184 root      20   0 4543356  74148  12960 S  80.0  40.9   0:19.96 java                                                          
22169 root      20   0 4543356  74148  12960 S  0.0  0.9   0:00.00 java                                                          
22170 root      20   0 4543356  74148  12960 S  0.0  0.9   0:00.35 java                                                          
22171 root      20   0 4543356  74148  12960 S  0.0  0.9   0:00.08 java                                                          
22172 root      20   0 4543356  74148  12960 S  0.0  0.9   0:00.09 java
```

将线程转为16进制

```text
$ printf "%x" 22184
56a8
```

使用`jstack`定位到线程

```text
$ jstack 22169 | grep 56a8
"Thread-1" #9 prio=5 os_prio=0 tid=0x00007fe428230800 nid=0x56a8 waiting on condition [0x00007fe4121a5000]
```

使用`3D`肉眼来查看线程代码

```text
"Thread-1" #9 prio=5 os_prio=0 tid=0x00007fe428230800 nid=0x56a8 waiting on condition [0x00007fe4121a5000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Unknown Source)
	at java.util.concurrent.TimeUnit.sleep(Unknown Source)
	at com.dounine.tool.http.sql.LogsRequest$4.run(LogsRequest.java:152)
	at java.lang.Thread.run(Unknown Source)
```

然后开始从`LogsRequest.java`152行开始找起，发现里面有一个死循环...

FIX它就好了

