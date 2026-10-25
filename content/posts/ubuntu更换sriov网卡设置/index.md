---
title: ubuntu更换sriov网卡设置
id: 331cf0ab-fe89-4ed2-a247-b61d48c1d696
date: 2025-09-29 15:00:41
auther: squaly
featured_image: Pasted%20image%2020261024075823.png
featured_image_preview: Pasted%20image%2020261024075823.png
summary: esxi先在虚拟交换机设置混杂模式，全部打开。然后开始检查ubuntu网卡工作情况。 1. 确认系统识别到网卡 lspci -nn | grep -i ether 看是否有 Mellanox/Intel/Broadcom 4121A 的 VF 或 PF 设备列出来。 SR-IOV VF 通常会标记
permalink: /archives/QVg5mSIh
categories:
  - network
tags:
  - ubuntu
---

esxi先在虚拟交换机设置混杂模式，全部打开。然后开始检查ubuntu网卡工作情况。

### 1. 确认系统识别到网卡

```bash
lspci -nn | grep -i ether
```

看是否有 **Mellanox/Intel/Broadcom 4121A** 的 VF 或 PF 设备列出来。
SR-IOV VF 通常会标记成 *Virtual Function*。

### 2. 查看网卡驱动是否加载

```bash
sudo lspci -nnk | grep -A3 -i 4121
```

检查 `Kernel driver in use` 是否正确（比如 `mlx5_core`、`ixgbevf`、`iavf`，取决于 4121A 芯片型号）。

### 3. 查看网卡接口

```bash
ip link show
```

确认有没有新增加的 `ethX` / `ensX` 网卡，状态是否是 `UP`。

### 4. 检查 dmesg 是否有报错

```bash
dmesg | grep -i eth
dmesg | grep -i vf
```

可以看到内核加载 VF 的情况，有时会报 "Failed to enable SR-IOV VF" 或 "Link Down"。

### 5. 驱动/固件情况

```bash
ethtool -i eth0
```

（把 `eth0` 换成对应的接口名）
查看驱动、版本、firmware 版本是否加载成功。

### 6. 网络连通性

如果已经配置 IP，可以测试：

```bash
ping -c 4 192.168.2.1
```

（换成你实际的网关地址）

### 7.手動把 `ens65` 啟用起來，並試著配置 IP：`ens65`改为你实际的数字

```bash
sudo ip link set ens65 up 
```

### 下一步你可以做的測試

#### 8. 確認是否有拿到 DHCP

你剛剛執行了 `dhclient ens65`，但沒有看到分配的 IP。檢查一下：

```bash
ip addr show ens65
```

如果沒有 IP，代表 DHCP server 沒回應。那就要改成靜態設置。

#### 9. 設置靜態 IP（例如 LAN: 10.10.10.252/24）

```bash
sudo ip addr add 10.10.10.252/24 dev ens65
sudo ip link set ens65 up
```

再確認：

```bash
ip addr show ens65
```

#### 10. 測試連線

假設你網段網關是 `19.168.2.1`，可以測試：

```bash
ping -c 4 192.168.2.1
```

如果能通，表示 VF 已經完全正常工作。


### 11.检查 Netplan 配置

Netplan 配置文件通常在：

```
/etc/netplan/01-netcfg.yaml
```

或者：

```
/etc/netplan/*.yaml
```

示例不正确配置可能是：

```yaml
network:
  version: 2
  ethernets:
    ens36:
      dhcp4: yes
    ens65:
      dhcp4: yes    # 这里应该改为静态
```

---

### 12.修改 Netplan 配置（正确示例）

配置如下：

```yaml
network:
  version: 2
  ethernets:
    ens36:
      dhcp4: yes
    ens65:
      dhcp4: no
      addresses:
        - 192.268.2.3/24
      gateway4: 192.168.2.1     # 替换为你实际网关
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
```

✅ 说明：

* **ens36**：保持 DHCP。
* **ens65**：禁用 DHCP，设置静态 IP、网关、DNS。
* 确保 **ens65** 的 IP 不会和 DHCP 池冲突，否则会报错或被路由器覆盖。

---

### 13.应用配置

保存文件后：

```bash
sudo netplan generate
sudo netplan apply
```

检查 IP：

```bash
ip a
ping 8.8.8.8
```

---

### 14.排查可能问题

1. **路由器 DHCP 冲突**
   如果路由器 DHCP 池包含 192.168.2.3，可能会覆盖静态 IP。建议把 192.168.2.3 从 DHCP 池里排除。
2. **Netplan 文件权限**
   Netplan 对权限比较敏感：

```bash
sudo chmod 644 /etc/netplan/01-netcfg.yaml
```

3. **Open vSwitch 警告**
   如果 ens65 被 OVS 管理，需要确保 OVS 没冲突，否则静态 IP 可能不会生效。
