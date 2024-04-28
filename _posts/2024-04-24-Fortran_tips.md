---
layout: post
title: "Fortran tips"
categories: Code
---

输出到流文件时指定`form='unformatted',access='stream'`如果文件原来存在可能不会覆盖文件末尾，需使用`status='replace'`但读取文件时不能用`status='replace'`<br/>
OpenMP的堆栈设置方法：`export OMP_STACKSIZE=64M`
使用Matlab Mexfunction调用Fortran时可以使用`call KMP_SET_STACKSIZE(100000000)`

Fortran计时函数：
```fortran
use omp_lib
real*8 time
time=omp_get_wtime()
```
```fortran
call system_clock(time)
!If the type is INTEGER(1), count_rate is 0. 
!If the type is INTEGER(2), count_rate is 1000. 
!If the type is INTEGER(4) or REAL(4), count_rate is 10000. 
!If the type is INTEGER(8), REAL(8), or REAL(16), count_rate is 1000000.
```
```fortran
real time
call cpu_time(time)
```