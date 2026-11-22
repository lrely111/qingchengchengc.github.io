---
title: "grep、sed、awk：文本处理三剑客"
date: 2026-11-12T19:30:00+08:00
draft: false
author: "2023902111"
categories: ["文本处理"]
tags: ["grep", "sed", "awk", "正则表达式"]
summary: "用一份 8 行的成绩单，把 grep 找行、sed 改行、awk 按字段统计这三种视角讲清楚，并给出管道组合的实际用法。"
ShowToc: true
---

## 一、三种视角

- **grep 看“行”**：按模式匹配，输出命中的行；
- **sed 改“行”**：按行执行编辑脚本（替换、删除、打印区间）；
- **awk 算“字段”**：默认按空白分列，擅长按列提取、条件过滤、数值统计。

数据是我实验二用的 `student_scores.txt`：

```text
zhang3 85 male ComputerScience
li4 92 female Geomatics
wang5 78 male Geology
zhao6 65 female Surveying
sun7 88 male ComputerScience
chen8 95 Male ComputerScience
huang9 70 Female Surveying
lin10 82 Male Geology
```

## 二、grep 的常用开关

```bash
grep ComputerScience student_scores.txt   # 基础匹配
grep -i "geo" student_scores.txt          # 忽略大小写
grep -n female student_scores.txt         # 显示行号
grep -v male student_scores.txt           # 反向匹配
grep -E "9[0-9]" student_scores.txt       # 扩展正则：90 分以上
grep -c Geology student_scores.txt        # 只输出匹配行数
```

其中 `-v` 与 `-i` 组合起来，就能做“排除某类记录”的粗筛。

## 三、sed 的替换与备份

```bash
sed 's/ComputerScience/CS/g' student_scores.txt        # 只打印，不改文件
sed -i 's/Male/Female/g' student_scores.txt            # 直接改
sed -i.bak 's/ComputerScience/CS/g' student_scores.txt # 改前自动备份为 .bak
sed -n '2,4p' student_scores.txt                       # 打印 2~4 行
sed '/Geology/d' student_scores.txt                    # 删除含 Geology 的行
```

经验：**先不带 `-i` 预览，确认无误再加 `-i`；重要文件一律用 `-i.bak`**。

## 四、awk 做统计

```bash
awk '{print $1, $2}' student_scores.txt
awk '$2>80 {print $1, $2}' student_scores.txt
awk '{sum+=$2} END {printf "总分: %.2f, 平均分: %.2f\n", sum, sum/NR}' student_scores.txt
awk '{sum[$4]+=$2; count[$4]++} END {for(i in sum) printf "%s: 人数=%d, 平均分=%.2f\n", i, count[i], sum[i]/count[i]}' student_scores.txt
```

得到总分 655、平均分 81.88；按专业分组时要注意 `for(i in sum)` 的输出顺序是不确定的，需要稳定顺序就加 `| sort`。

## 五、管道：把工具串起来

```bash
grep CS student_scores.txt | awk '{print $1, $2}'
sort -k2 -n student_scores.txt | tail -3
awk '{print $4}' student_scores.txt | sort | uniq -c
grep -E "8[0-9]|9[0-9]" student_scores.txt > high_scores.txt
```

管道把“过滤 → 取列 → 排序 → 统计 → 落盘”串成一条命令，比写脚本轻量得多。
写日志分析脚本时，这三件工具基本能覆盖 90% 的需求。
