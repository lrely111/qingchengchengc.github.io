---
title: "Makefile 实践：从基础版到增强版"
date: 2026-11-18T20:40:00+08:00
draft: false
author: "王文龙"
categories: ["构建工具"]
tags: ["make", "gcc", "编译", "自动化"]
summary: "用一个 hello 程序，对比基础版、变量版、增强版三个 Makefile，理解目标、依赖、自动变量和 .PHONY 的作用。"
ShowToc: true
---

## 一、gcc 的四个阶段

`gcc` 把 `.c` 变成 ELF 可执行文件要经过四步：

1. 预处理（`-E`）：展开 `#include`、替换宏，得到 `hello.i`；
2. 编译（`-S`）：生成汇编 `hello.s`；
3. 汇编（`-c`）：生成可重定位目标文件 `hello.o`；
4. 链接：合并目标文件与库，生成 `hello`。

一条 `gcc hello.c -o hello` 内部仍然走这四步。

## 二、基础版 Makefile

```make
hello: main.o hello.o
	gcc -o hello main.o hello.o

main.o: main.c hello.h
	gcc -c main.c

hello.o: hello.c hello.h
	gcc -c hello.c

clean:
	rm -f hello main.o hello.o

.PHONY: clean
```

**命令行必须用 Tab 缩进**，用空格会报 `*** missing separator`，这是新手最常踩的坑。

## 三、变量版

```make
CC = gcc
CFLAGS = -Wall -g -O2
OBJS = main.o hello.o

hello: $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c hello.h
	$(CC) $(CFLAGS) -c $< -o $@
```

自动变量：`$@` 当前目标、`$^` 全部依赖、`$<` 第一个依赖。改编译器或优化等级时只改一行，规则可复用。

## 四、增强版

```make
all: hello

run: hello
	./hello

clean:
	rm -f hello $(OBJS)

distclean: clean
	rm -f *.bak.*
	rm -rf backup_logs/

depends:
	@echo "源文件: $(SRCS)"

.PHONY: all run clean distclean depends help
```

`make help` 把可用目标列出来，`make run` 编译并运行，`make distclean` 连备份和日志一起清掉。
`.PHONY` 声明伪目标，避免当前目录恰好存在同名文件时 make 认为“已是最新”。

## 五、小结

Makefile 的核心只有一句话：**按“目标-依赖”的时间戳决定要不要执行命令**。
变量和自动变量让规则可复用，`.PHONY` 保证伪目标每次都执行，Tab 缩进和 `.PHONY` 是最容易出错的两处。
再往上走就是 CMake / Meson / Ninja，思路仍是“描述依赖 + 增量构建”。
