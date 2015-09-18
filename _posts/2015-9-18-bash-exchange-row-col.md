---
layout: post
category: linux
title: Bash 行列转换小技巧
---

## 前言
有时我们监控系统性能的时候经常回去采集系统状态，并记录到文件中。  
然而有时候系统的某些状态信息是行列颠倒的，比如：

```bash
$ cat /proc/2483/status
...
VmPeak:	   28184 kB
VmSize:	   28120 kB
VmLck:	       0 kB
VmPin:	       0 kB
VmHWM:	    5292 kB
VmRSS:	    5260 kB
VmData:	    3224 kB
VmStk:	     136 kB
VmExe:	     956 kB
VmLib:	    2288 kB
VmPTE:	      80 kB
VmSwap:	       0 kB
...
```

这个时候我们就需要把他给转回来！

## Trick

```bash
#!/bin/bash
head=$(cat /proc/2483/status | grep 'Vm' | awk -F ': *' '{print $1}')
body=$(cat /proc/2483/status | grep 'Vm' | awk -F ': *' '{print $2}')
echo $head
echo $body
```

Output

```bash
VmPeak VmSize VmLck VmPin VmHWM VmRSS VmData VmStk VmExe VmLib VmPTE VmSwap
28184 kB 28120 kB 0 kB 0 kB 5292 kB 5260 kB 3224 kB 136 kB 956 kB 2288 kB 80 kB 0 kB
```
