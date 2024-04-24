---
layout: post
title: "ifort/gfortran差异"
categories: Code
---

## 自带函数
gfortran中一些自带函数不可用，如sind，需要开启GNU Extension<br />`ifort = gfortran -fdec-math`
## 代码列数限制
gfortran限制代码长度，需设置为无限制<br />`ifort = gfortran -ffree-line-length-none`
## 无格式文件直接读取
`recl`取值不同，ifort中单位为4字节，gfortran为1字节<br />`ifort -assume byterecl = gfortran`<br />程序中可查询数据所占`recl`：<br />`inquire(iolength=reclen) recltest`<br />[https://www.likecs.com/ask-1149743.html#sc=1087.5](https://www.likecs.com/ask-1149743.html#sc=1087.5)

