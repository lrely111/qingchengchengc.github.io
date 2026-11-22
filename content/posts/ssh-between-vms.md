---
title: "两台虚拟机之间配置 SSH 免密登录"
date: 2026-11-05T19:20:00+08:00
draft: false
author: "2023902111"
categories: ["Linux基础"]
tags: ["ssh", "virtualbox", "网络"]
summary: "记录 VirtualBox 里两台 Ubuntu 虚拟机的组网（NAT + 内部网络）、克隆后 IP 冲突的排查，以及 ssh-keygen 与 ssh-copy-id 配免密的完整过程。"
ShowToc: true
---

## 一、组网方式

虚拟机里每台机器两块网卡：

- `enp0s3`：NAT，用于上网，IP 是 10.0.2.15；
- `enp0s8`：内部网络 `linuxlab`，两台在同一个二层网络里，手工配静态 IP。

```yaml
# /etc/netplan/99-lab.yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s8:
      dhcp4: false
      addresses: [192.168.100.101/24]
      optional: true
```

`node1` 用 .101，`node2` 用 .102，`netplan apply` 后 `ip -4 addr show enp0s8` 确认。

## 二、克隆后的“IP 一样”问题

第一次克隆虚拟机后，两台机器 `ip addr` 完全相同、`ssh` 连不上，甚至提示主机指纹变了。
原因是**网卡的 MAC 地址被一起克隆了**：DHCP 服务器按 MAC 分配租约，MAC 相同就拿到同一个 IP。

解决办法：

1. 克隆/导入时勾选“重新生成 MAC 地址”；
2. 给克隆出来的机器配置不同的静态 IP；
3. 重装或克隆后如果连旧机器，执行 `ssh-keygen -R 对端IP` 清掉旧指纹。

## 三、口令登录与免密

```bash
ssh student@192.168.100.102      # 首次会问 yes/no，输入 yes 后输密码
hostname                          # 对端执行，确认已登录 node2
exit
```

免密登录三步：

```bash
ssh-keygen -t rsa -b 2048 -N "" -f ~/.ssh/id_rsa   # 生成密钥对
ssh-copy-id student@192.168.100.102                # 推送公钥到对端
ssh student@192.168.100.102 'hostname'             # 不再询问密码
```

原理是对端用 `authorized_keys` 里的公钥加密一段随机串，只有持有私钥的一方能解开，私钥始终不离开本机。

## 四、排错顺序

按“地址 → 链路 → 服务 → 策略”查：

1. 两台是否在同一网段、IP 是否冲突；
2. `ping` 是否通，网卡是否 UP；
3. 对端 `systemctl status ssh` 是否 active；
4. 防火墙是否放行 22（`ufw allow 22` 或 `firewall-cmd --add-service=ssh`）；
5. CentOS 还要看 SELinux 是否拦截。

这套排查顺序后来在实验里反复用到，比“重启试试”有效得多。
