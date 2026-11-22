---
title: "Linux 目录结构与文件权限"
date: 2026-11-08T20:10:00+08:00
draft: false
author: "2023902111"
categories: ["Linux基础"]
tags: ["linux", "文件系统", "权限"]
summary: "从根目录 / 开始理解 Linux 的树形目录结构，以及 rwx 权限位在文件和目录上的不同含义。"
cover:
  image: ""
  alt: ""
  caption: ""
ShowToc: true
TocOpen: false
---

## 一、为什么要先理解目录结构

Linux 只有一棵以 `/` 为根的树，所有分区、U 盘、光盘都挂载到某个目录上，没有 Windows 的 C:、D: 盘符。
初学者最容易踩的坑是把文件写到了“以为在”的地方，所以先弄清楚这几个目录：

| 目录 | 作用 |
| --- | --- |
| `/etc` | 系统与服务的配置文件，如 `/etc/passwd`、`/etc/ssh/sshd_config` |
| `/home` | 普通用户主目录，我的账号是 `/home/student` |
| `/root` | root 的主目录 |
| `/bin`、`/usr/bin` | 可执行命令（Ubuntu 22.04 中 `/bin` 是 `/usr/bin` 的软链接） |
| `/var/log` | 日志目录，排查服务问题时最先看这里 |
| `/tmp` | 临时文件，所有用户可写，重启可能被清空 |

用 `ls -ld /bin` 就能看到 `/bin -> usr/bin` 的软链接关系，这也是 `du -sh /bin` 显示 0 的原因，要加 `-L` 才会跟进统计。

## 二、权限位：rwx 到底控制什么

`ls -l` 的第一列 `-rwxr-xr--` 分三组：属主（u）、属组（g）、其他人（o），每组三位 rwx。

- 对**文件**：r 可读内容、w 可改内容、x 可执行；
- 对**目录**：r 可列出目录内容、w 可在目录内增删文件、x 可进入目录（cd）或访问其中文件。

一个容易忽略的细节：**目录没有 x 权限时，即使文件本身可读也打不开**，因为无法“穿过”目录。

## 三、实验里踩过的两个坑

1. `mkdir -p file_experiment/{dir1, dir2, dir3}` 里逗号后多了空格，Bash 会把它当成三个独立的词，建出 `dir1,` 这种带逗号的目录。正确写法是 `{dir1,dir2,dir3}`。
2. 用 `chmod u-r a.txt` 去掉属主读权限后，`cat` 报 `Permission denied`，`./a.txt` 直接执行也会失败——因为脚本解释器需要先读取文件内容。恢复用 `chmod u+rw a.txt`。

## 四、小结

记住三句话：**一切皆文件、权限分三组、路径分绝对与相对**。
`pwd` 确认位置、`ls -l` 确认权限、`chmod/chown` 修改属性，这三步能解决 80% 的文件相关报错。
